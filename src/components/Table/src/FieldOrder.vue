<template>
  <div class="k-table__filed-order" @click="changeOrder">
    <k-icon
      :size="16"
      type="sort"
      class="k-table__filed-order-default"
      :class="{'k-table__filed-order--asc': order === 'asc', 'k-table__filed-order--desc': order === 'desc'}"
      ></k-icon>
  </div>
</template>
<script>
import KIcon from '@/components/Icon'
import { inject, defineComponent } from 'vue'
const orders = ['', 'desc', 'asc']
function nextOrder(order) {
  const index = orders.findIndex((item) => item === order)
  return orders[(index + 1)%3]
}
export default defineComponent({
  name: 'KFieldOrder',
  props: {
    column: {
      type: Object,
      required: true
    },
    order: {
      type: String,
      default: ''
    }
  },
  setup(props) {
    const tableEmit = inject('tableEmit');
    const columnStore = inject('columnStore')
    const changeOrder = () => {
      const order = nextOrder(props.order)
      columnStore.action.updateColumnOrder(props.column.id, order)
      tableEmit('order-change', props.column, order)
    }
    return {
      changeOrder
    }
  },
  components: {
    KIcon,
  }
})
</script>
<style>
.k-table__filed-order {
  @apply flex justify-center items-center cursor-pointer;
}
.k-table__filed-order--asc {
  background: linear-gradient(#27272a 50%, #a1a1aa 50%)
}
.k-table__filed-order--desc {
  background: linear-gradient(#a1a1aa 50%, #27272a 50%)
}
.k-table__filed-order-default {
  background-color: #a1a1aa;
}
</style>