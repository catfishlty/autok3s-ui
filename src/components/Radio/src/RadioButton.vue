<template>
  <label
    class="k-radio-button"
    :class="[model === label ? 'bg-light-blue-600' : 'bg-gray-200', isDisabled ? 'disabled' : '']">
    <input
      ref="radioRef"
      type="radio"
      v-model="model"
      :value="label"
      :name="name"
      :disabled="isDisabled"
      class="k-radio-button__origin-radio">
    <span
      class="k-radio-button__label flex items-center"
    >
      <slot>{{label}}</slot>
    </span>
  </label>
</template>
<script>
import {computed, defineComponent, inject, ref} from 'vue'
export default defineComponent({
  name: 'KRadioButton',
  props: {
    modelValue: {
      type: [String, Number, Boolean],
      default: '',
    },
    label: {
      type: [String, Number, Boolean],
      default: '',
    },
    name: {
      type: String,
      default: '',
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  emits: ['update:modelValue'],
  setup(props, {emit}) {
    const radioRef = ref(null)
    const group = inject('radioGroup')
    const model = computed({
      get() {
        return group?.modelValue ?? props.modelValue
      },
      set(v) {
        if (group) {
          group.changeEvent(v)
        } else {
          emit('update:modelValue', v)
        }
        radioRef.value.checked = props.modelValue === props.label
      }
    })
    const isDisabled = computed(() => {
      return group?.disabled || props.disabled
    })
    return {
      model,
      isDisabled,
      radioRef,
    }
  }
})
</script>
<style>
.k-radio-button {
  display: grid;
  grid-template-areas: 'a';
  align-items: center;
  justify-items: center;
  @apply px-21px leading-40px;
}

.k-radio-button:not(.disabled) {
  cursor: pointer;
}
.k-radio-button__origin-radio {
  opacity: 0;
}
.k-radio-button__origin-radio, .k-radio-button__label {
  grid-area: a;
}

</style>
