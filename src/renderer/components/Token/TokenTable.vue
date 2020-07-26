<template>
  <div class="TokenTable w-full">
    <TableWrapper
      v-bind="$attrs"
      :columns="columns"
      :no-data-message="$t('TABLE.NO_TOKENS')"
      v-on="$listeners"
      @on-sort-change="onSortChange"
    >
      <template
        slot-scope="data"
      >
        <div
          v-if="data.column.field === 'name'"
        >
          {{ data.row.name }}
        </div>

        <div
          v-else-if="data.column.field === 'symbol'"
        >
          {{ data.row.symbol }}
        </div>

        <div
          v-else-if="data.column.field === 'id'"
        >
          <a
            class="flex items-center whitespace-no-wrap"
            href="#"
            @click.stop="network_openExplorer('token', data.row.tokenIdHex)"
          >
            <span
              class="mr-1"
            >
              {{ data.row.tokenIdHex }}
            </span>
            <SvgIcon
              name="open-external"
              view-box="0 0 12 12"
              class="text-theme-page-text-light"
            />
          </a>
        </div>

        <div
          v-else-if="data.column.field === 'balance'"
          class="flex items-center justify-end"
        >
          <span
            class="font-bold mr-2 whitespace-no-wrap"
          >
            {{ currency_decimals(data.row.tokenBalance, data.row.tokenDecimals) }}
          </span>
        </div>
      </template>
    </TableWrapper>

    <Portal
      v-if="selected"
      to="modal"
    >
      <TransactionShow
        :transaction="selected"
        @close="onCloseModal"
      />
    </Portal>
  </div>
</template>

<script>
import SvgIcon from '@/components/SvgIcon'
import truncateMiddle from '@/filters/truncate-middle'
import TableWrapper from '@/components/utils/TableWrapper'

export default {
  name: 'TokenTable',

  components: {
    SvgIcon,
    TableWrapper
  },

  props: {
    hasShortId: {
      type: Boolean,
      required: false,
      default: false
    },
    isDashboard: {
      type: Boolean,
      required: false,
      default: false
    }
  },

  data: () => ({
    selected: null
  }),

  computed: {
    columns () {
      const vendorFieldClass = [
        'hidden', 'w-1/4'
      ]
      if (this.hasShortId && !this.isDashboard) {
        vendorFieldClass.push('xxl:table-cell')
      } else if (!this.isDashboard) {
        vendorFieldClass.push('xl:table-cell')
      }

      return [
        {
          label: this.$t('WALLET_TOKENS.NAME'),
          field: 'name'
        },
        {
          label: this.$t('WALLET_TOKENS.SYMBOL'),
          field: 'symbol'
        },
        {
          label: this.$t('WALLET_TOKENS.ID'),
          field: 'id',
          formatFn: this.formatTokenId
        },
        {
          label: this.$t('WALLET_TOKENS.BALANCE'),
          type: 'number',
          field: 'balance',
          formatFn: this.formatBalance,
          tdClass: 'text-right',
          thClass: 'text-right'
        }
      ]
    }
  },

  methods: {
    formatDate (value) {
      return this.formatter_date(value)
    },

    formatTokenId (value) {
      return this.hasShortId ? truncateMiddle(value, 6) : truncateMiddle(value, 10)
    },

    formatBalance (value) {
      return this.formatter_networkCurrency(value)
    },

    openTokens (id) {
      this.network_openExplorer('token', id)
    },

    onSortChange ({ columnIndex, sortType }) {
      if (this.columns[columnIndex]) {
        const columnName = this.columns[columnIndex].field
        this.$emit('on-sort-change', { columnName, sortType })
      }
    },

    onRowClick ({ row }) {
      this.selected = row
    },

    onCloseModal () {
      this.selected = null
    }
  }
}
</script>

<style lang="postcss">

</style>
