<template>
<div>
  <div class="grid grid-cols-2 gap-10px">
    <k-select
      v-model="provider"
      label="Provider"
      required
      :loading="loading"
    >
      <k-option
        v-for="p in providerOptions"
        :label="p.name"
        :value="p.id"
        :key="p"></k-option>
    </k-select>
    <div></div>
    <template v-for="[p, v] in Object.entries(providerKeyFieldMap).filter((e) => e[0] !== 'google')"
      :key="p"
      >
      <k-password-input
        v-model.trim="form[p].key"
        :label="form[p].keyLabel"
        required
        v-show="p === provider"
      />

      <k-password-input
        v-model.trim="form[p].secret"
        :label="form[p].secretLabel"
        required
        v-show="p === provider"
      />
    </template>
    <string-form
      v-model.trim="form.google.key"
      :label="form.google.keyLabel"
      required
      v-show="'google' === provider"
    />
    <string-form
      v-model.trim="form.google.secret"
      :label="form.google.secretLabel"
      required
      v-show="'google' === provider"
    />
  </div>
  <footer-actions>
    <router-link :to="{name: 'ClusterExplorerSettingsCredentials'}" class="btn role-secondary">Cancel</router-link>
    <k-button class="role-primary" :loading="loading" @click="create">Create</k-button>
  </footer-actions>
  <k-alert v-for="(e, index) in errors" :key="index" type="error" :title="e"></k-alert>
</div>
</template>
<script>
import {computed, defineComponent, reactive, ref, watchEffect} from 'vue'
import { useRouter } from 'vue-router'
import {startCase} from 'lodash-es'
import useProviderKeyMap from '../composables/useProviderKeyMap.js'
import useProviders from '@/composables/useProviders.js'
import useCredentials from '@/composables/useCredentials.js'
import { createCredential } from '@/api/credential';
import FooterActions from '@/views/components/FooterActions.vue'
import {stringify} from '@/utils/error.js'
import StringForm from '@/views/components/baseForm/StringForm.vue'

export default defineComponent({
  setup() {
    const router = useRouter()
    const {providerKeyFieldMap} = useProviderKeyMap()
    const {loading: providersLoading, error: providerError, providers} = useProviders()
    const {loading: credentialLoading, error: credentialError, credentials} = useCredentials()
    const creating = ref(false)
    const form = reactive(Object.entries(providerKeyFieldMap)
      .reduce((t, [k, v]) => {
        t[k] = {
            keyLabel: startCase(v.key),
            secretLabel: startCase(v.secret),
            key: '',
            secret: '',
          }
        return t
      }, {}))
    const provider = ref(null)
    const formErrors = ref([])
    const loading = computed(() => {
      return creating.value || providersLoading.value || credentialLoading.value
    })
    const errors = computed(() => {
      const errors = []
      if (providerError.value) {
        errors.push(stringify(providerError))
      }
      if (credentialError.value) {
        errors.push(stringify(credentialError))
      }
      errors.push(...formErrors.value)
      return errors
    })
    const providerOptions = computed(() => {
      const providersWithCredential = Object.keys(providerKeyFieldMap)
      const credencialProviders = credentials.value
        .reduce((t, c)=> {
          if (!t.includes(c.provider) && providersWithCredential.includes(c.provider)) {
            t.push(c.provider)
          }
          return t
        }, [])
      return providers.value.filter((p) => providersWithCredential.includes(p.id) && !credencialProviders.includes(p.id))
    })
    watchEffect(() => {
      if (!loading.value && providerOptions.value.length > 0 && !provider.value) {
        provider.value=providerOptions.value[0]?.id
      }
    })
    const validate = () => {
      const errors = []
      const {key, secret, keyLabel, secretLabel} = form[provider.value]
      if (!key) {
        errors.push(`"${keyLabel}" is required`)
      }
      if (!secret) {
        errors.push(`"${secretLabel}" is required`)
      }
      formErrors.value = errors
      return errors.length === 0
    }
    const create = async () => {
      if (!validate()) {
        return
      }
      creating.value = true
      const { key, secret } = form[provider.value]
      const postData = {
        provider: provider.value,
        secrets: {
          [providerKeyFieldMap[provider.value].key]: key,
          [providerKeyFieldMap[provider.value].secret]: secret
        }
      }
      try {
        await createCredential(postData)
      } catch (err) {
        formErrors.value = [stringify(err)]
      }
      creating.value = false
      router.push({name: 'ClusterExplorerSettingsCredentials'})
    }
    return {
      form,
      provider,
      providerOptions,
      loading,
      errors,
      startCase,
      create,
      providerKeyFieldMap,
    }
  },
  components: {
    FooterActions,
    StringForm,
  }
})
</script>
