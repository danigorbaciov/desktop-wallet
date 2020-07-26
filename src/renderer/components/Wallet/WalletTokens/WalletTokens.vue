<template>
  <div class="WalletTokens">
    <TokenTable
      :current-page="currentPage"
      :rows="fetchedTokens"
      :total-rows="totalCount"
      :is-loading="isLoading"
      :is-remote="true"
      :has-pagination="totalCount > 0"
      :sort-query="queryParams.sort"
      :per-page="tokenTableRowCount"
      @on-per-page-change="onPerPageChange"
      @on-page-change="onPageChange"
      @on-sort-change="onSortChange"
    />
  </div>
</template>

<script>
import { at, clone } from 'lodash'
// import mergeTableTokens from '@/components/utils/merge-table-tokens'
import TokenTable from '@/components/Token/TokenTable'
import QreditSltService from '@/services/qreditslt'

export default {
  name: 'WalletTokens',

  components: {
    TokenTable
  },

  data: () => ({
    currentPage: 1,
    isFetching: false,
    isLoading: false,
    fetchedTokens: [],
    expiryEvents: [],
    totalCount: 0,
    lastStatusRefresh: null,
    queryParams: {
      page: 1,
      limit: 10,
      sort: {
        field: 'timestamp',
        type: 'desc'
      }
    }
  }),

  computed: {
    tokenTableRowCount: {
      get () {
        return this.$store.getters['session/tokenTableRowCount']
      },
      set (count) {
        this.$store.dispatch('session/setTokenTableRowCount', count)
        const profile = clone(this.session_profile)
        profile.tokenTableRowCount = count
        this.$store.dispatch('profile/update', profile)
      }
    }
  },

  watch: {
    // This watcher would invoke the `fetch` after the `Synchronizer`
    wallet_fromRoute (newValue, oldValue) {
      // const currentTimestamp = Math.round((new Date()).getTime() / 1000)
      if (newValue.address !== oldValue.address) {
        this.reset()
        this.loadTokens()
      }
    }
  },

  created () {
    this.loadTokens()
    this.$eventBus.on('wallet:reload', this.loadTokens)
  },

  beforeDestroy () {
    this.$eventBus.off('wallet:reload', this.loadTokens)
  },

  methods: {

    getStoredTokens (address) {
      if (!address) {
        return []
      }

      // return this.$store.getters['tokens/byAddress'](address, { includeExpired: true })
    },

    async getTokens (address) {
      if (!address) {
        return []
      }

      const { limit, page, sort } = this.queryParams

      return QreditSltService.getAllWalletTokens(address, {
        page,
        limit,
        orderBy: `${sort.field}:${sort.type}`
      })
    },

    async fetchTokens () {
      // If we're already fetching, it's unneccessary to fetch again
      if (this.isFetching) {
        return
      }

      let address
      if (this.wallet_fromRoute) {
        address = this.wallet_fromRoute.address.slice()
      }

      this.isFetching = true

      try {
        const tokens = await this.getTokens(address)

        // this.$store.dispatch('token/deleteBulk', {
        //   tokens: response.tokens,
        //   profileId: this.session_profile.id
        // })

        // const tokens = mergeTableTokens(response.tokens, this.getStoredTokens(address))

        if (this.wallet_fromRoute && address === this.wallet_fromRoute.address) {
          this.$set(this, 'fetchedTokens', tokens)
          // this.totalCount = response.totalCount
        }
      } catch (error) {
        // Ignore the 404 error of wallets that are not on the blockchain
        const messages = at(error, 'response.data.message')
        if (messages[0] !== 'Wallet not found') {
          this.$logger.error(error)

          this.$error(this.$t('COMMON.FAILED_FETCH', {
            name: 'tokens',
            msg: error.message
          }))
        }
        this.fetchedTokens = []
      } finally {
        this.isFetching = false
        this.isLoading = false
      }
    },

    /**
     * Fetch the transaction and show the loading animation while the response
     * is received
     */
    async loadTokens () {
      if (!this.wallet_fromRoute || this.isFetching) {
        return
      }

      this.newTransactionsNotice = null
      this.isLoading = true
      this.fetchTokens()
    },

    onPageChange ({ currentPage }) {
      this.currentPage = currentPage
      this.__updateParams({ page: currentPage })
      this.loadTokens()
    },

    onPerPageChange ({ currentPerPage }) {
      this.__updateParams({ limit: currentPerPage, page: 1 })
      this.loadTokens()
      this.tokenTableRowCount = currentPerPage
    },

    onSortChange (sortOptions) {
      const columnName = sortOptions[0].field
      const sortType = sortOptions[0].type

      this.__updateParams({
        sort: {
          field: columnName,
          type: sortType
        },
        page: 1
      })
      this.loadTokens()
    },

    reset () {
      this.currentPage = 1
      this.queryParams.page = 1
      this.totalCount = 0
      this.fetchedTokens = []
    },

    __updateParams (newProps) {
      this.queryParams = Object.assign({}, this.queryParams, newProps)
    }
  }
}
</script>
