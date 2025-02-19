<template>
  <div class="k-input" :class="{ disabled: disabled }">
    <div class="k-input__label">
      <label v-if="label" :for="inputId">{{label}} <sup v-if="required" class="text-red-500 top-0">*</sup></label>
      <tooltip v-if="desc">
        <k-icon type="prompt"></k-icon>
        <template #popover>{{desc}}</template>
      </tooltip>
    </div>
    <div class="k-input__prefix" v-if="$slots.prefix">
      <slot name="prefix">{{prefix}}</slot>
    </div>
    <textarea v-if="type==='textarea'"
      class="k-input__textarea"
      :class="[!label ? 'py-10.5px px-0' : '']"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      :id="inputId"
      :disabled="disabled"
      :autocomplete="autocomplete"
      :rows="rows"
      v-bind="$attrs"
      ref="inputRef"
    ></textarea>
    <input
      v-else
      class="k-input__input"
      :class="[!label ? 'py-10.5px px-0' : '']"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      :id="inputId"
      :disabled="disabled"
      :autocomplete="autocomplete"
      :type="type"
      v-bind="$attrs"
      ref="inputRef">
    <div class="k-input__suffix" v-if="$slots.suffix">
      <slot name="suffix"></slot>
    </div>
  </div>
</template>
<script>
import {ref, defineComponent} from 'vue'
import Tooltip from '@/components/Tooltip'
import KIcon from '@/components/Icon'
import {useIdGenerator} from '@/utils/idGenerator.js'
const getId = useIdGenerator(0, 'labeled-input_');
export default defineComponent({
  name: 'KInput',
  inheritAttrs: false,
  props: {
    label: {
      type: String,
      default: ''
    },
    required: {
      type: Boolean,
      default: false,
    },
    modelValue: {
      type: [String, Boolean, Number]
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    desc: {
      type: String,
      default: ''
    },
    autocomplete: {
      type: String,
      default: 'off'
    },
    type: {
      type: String,
      default: 'text'
    },
    rows: {
      type: Number,
      default: 2
    }
  },
  emits: ['update:modelValue'],
  setup() {
    const inputId = getId()
    const inputRef = ref(null)
    const focus = () => {
      inputRef.value?.focus();
    }
    return {
      inputId,
      inputRef,
      focus,
    }
  },
  components: {
    Tooltip,
    KIcon,
  }
})
</script>
<style>
.k-input {
  display: grid;
  grid-template-areas: "label label suffix"
                       "prefix input suffix";
  grid-template-columns: auto 1fr auto;
  grid-template-rows: auto 1fr;
  @apply p-8px rounded border border-gray-300;
}
.k-input:not(.disabled):hover {
  @apply bg-gray-100;
}
.k-input__label {
  grid-area: label;
  @apply grid gap-x-10px items-center grid-cols-[max-content,auto] text-warm-gray-500;
  width: fit-content;
}
.k-input__prefix {
  grid-area: prefix;
  @apply text-warm-gray-500 self-center;
}
.k-input__suffix {
  grid-area: suffix;
  @apply flex items-center text-warm-gray-500 pl-8px border-l border-warm-gray-600 font-400;
}
input.k-input__input, .k-input__textarea,
input.k-input__input:hover, .k-input__textarea:hover,
input.k-input__input:focus, .k-input__textarea:focus {
  grid-area: input;
  @apply bg-transparent focus-visible:outline-none;
}
</style>
