<template>
  <div class="k-alert" v-show="visible" :class="typeClass">
    <div class="k-alert__title"><slot>{{title}}</slot></div>
    <div class="k-alert__close" v-if="closable"><k-icon type="close"></k-icon></div>
    <div class="k-alert-desc" v-if="description">{{description}}</div>
  </div>
</template>
<script>
import {computed, defineComponent, ref} from 'vue'
import KIcon from '@/components/Icon'
export default defineComponent({
  name: 'KAlert',
  props: {
    title: {
      type: String,
      default: '',
    },
    closable: {
      type: Boolean,
      default: false
    },
    description: {
      type: String,
      default: ''
    },
    type: {
      type: String,
      default: 'success'
    }
  },
  emits: ['close'],
  setup(props, {emit}) {
    const visible = ref(true)

    const close = (e) => {
      visible.value = false
      emit('close', e)
    }
    const typeClass = computed(() => `k-alert--${props.type}`)
    return {
      visible,
      close,
      typeClass,
    }
  },
  components: {
    KIcon
  }
})
</script>
<style>
.k-alert {
  display: grid;
  grid-template-areas: "title close"
                       "desc desc";
  grid-template-columns: 1fr 28px;
  align-items: center;
  @apply p-10px my-15px;
}
.k-alert__title {
  grid-area: title;
}
.k-alert__close {
  grid-area: close;
}
.k-alert__desc {
  grid-area: desc;
}

.k-alert--success {
  @apply bg-$success-banner-bg border-l-4px border-$success;
}

.k-alert--info {
  @apply bg-$info-banner-bg border-l-4px border-$info;
}

.k-alert--warning {
  @apply bg-$warning-banner-bg border-l-4px border-$warning;
}

.k-alert--error {
  @apply bg-$error-banner-bg border-l-4px border-$error;
}
</style>