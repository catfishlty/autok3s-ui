<template>
  <teleport to="body">
    <div class="flex flex-col items-center justify-center z-$modal-z-index absolute top-0 left-0 bottom-0 right-0 bg-[rgba(0,0,0,.5)]" v-if="visible">
      <div class="min-w-1/3 bg-white">
        <div class="flex justify-between items-center pt-20px pb-10px px-20px">
          <div class="text-18px">
            <slot name="title">
              <span>{{title}}</span>
            </slot>
          </div>
          <k-icon v-if="showClose" type="close" class="cursor-pointer" @click="close"></k-icon>
        </div>
         <hr>
        <div class="py-30px px-20px">
          <slot></slot>
        </div>
        <div class="py-10px px-20px grid grid-flow-col gap-10px items-center justify-end">
          <slot name="footer"></slot>
        </div>
      </div>
    </div>
  </teleport>
</template>
<script>
import { ref, watchEffect, defineComponent } from 'vue'
import KIcon from '@/components/Icon'
export default defineComponent({
  name: 'KModal',
  props: {
    modelValue: {
      type: Boolean,
      required: true,
    },
    showClose: {
      type: Boolean,
      default: false,
    }
  },
  emits: ['update:modelValue'],
  setup(props, context) {
    const {emit} = context
    const visible = ref(false)
    watchEffect(() => {
      visible.value = props.modelValue
    })
    const close = () => {
      emit('update:modelValue', false)
    }
    return {
      visible,
      close,
    }
  },
  components: {
    KIcon,
  }
})
</script>
