<template>
  <dropdown-menu-item
    :class="[currentValue.includes(value) ? 'text-white bg-warm-gray-400' : 'text-light-blue-500']"
    @click="setValue">
    {{label ?? value}}
  </dropdown-menu-item>
</template>
<script>
import {defineComponent, inject, getCurrentInstance, onBeforeUnmount, computed, onMounted} from 'vue'
import { DropdownMenuItem }from '@/components/Dropdown'
export default defineComponent({
  name: 'KOption',
  props: {
    label: {
      type: String,
      required: true
    },
    value: {
      type: [String, Number, Boolean],
      required: true
    }
  },
  setup() {
    const selectStore = inject('selectStore')
    const multiple = inject('multiple')
    if (!selectStore) {
      console.warn('selectStore not provide')
      return
    }
    const currentOption = getCurrentInstance().proxy
    selectStore.action.addOption(currentOption)
    onBeforeUnmount(() => {
      selectStore.action.removeOption(currentOption)
    })
    const currentValue = computed(() => {
      if (multiple) {
        return selectStore.state.values ?? []
      }
      return [selectStore.state.value]
    })
    const setValue = (e) => {
      if (multiple) {
        e.stopPropagation()
        selectStore.action.setValues([currentOption.value])
      } else {
        selectStore.action.setValue(currentOption.value)
      }
      
    }
    return {
      currentValue,
      setValue,
    }
  },
  components: {
    DropdownMenuItem,
  }
})
</script>
