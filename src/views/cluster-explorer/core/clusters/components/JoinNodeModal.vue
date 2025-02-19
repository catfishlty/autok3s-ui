<template>
<k-modal v-model="visible">
  <template #title>Join Node</template>
  <template #default>
  <k-loading :loading="loading">
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-10px">
      <div>Desired Master Nodes: {{desiredNodes.master}}</div>
      <div>Desired Worker Nodes: {{desiredNodes.worker}}</div>
      <template v-if="provider === 'native'">
        <ip-address-pool-form
          ref="masterIps"
          v-model="nativeForm['master-ips']"
          label="Master IPs"
          :desc="nativeProviderSchema?.options?.['master-ips']?.description"
        ></ip-address-pool-form>
        <ip-address-pool-form
          ref="workerIps"
          v-model="nativeForm['worker-ips']"
          label="Worker IPs"
          :desc="nativeProviderSchema?.options?.['worker-ips']?.description"
        ></ip-address-pool-form>
        <string-form
          v-model.trim="nativeForm['ssh-user']"
          label="SSH User"
          :desc="nativeProviderSchema?.config?.['ssh-user']?.description"
        />
        <string-form
          v-model.trim="nativeForm['ssh-port']"
          label="SSH Port"
          :desc="nativeProviderSchema?.config?.['ssh-port']?.description"
        />
        <string-form
          v-model.trim="nativeForm['ssh-key-path']"
          label="SSH Key Path"
          :desc="nativeProviderSchema?.config?.['ssh-key-path']?.description"
        />
        <k-password-input
          v-model.trim="nativeForm['ssh-key-passphrase']"
          label="SSH Key Passphrase"
          :desc="nativeProviderSchema?.config?.['ssh-key-passphrase']?.description"
        />
        <string-form
          v-model.trim="nativeForm['ssh-cert-path']"
          label="SSH Cert Path"
          :desc="nativeProviderSchema?.config?.['ssh-cert-path']?.description"
        />
        <k-password-input
          v-model.trim="nativeForm['ssh-password']"
          label="SSH Password"
          :desc="nativeProviderSchema?.config['ssh-password']?.description"
        />
        <boolean-form
          v-model="nativeForm['ssh-agent-auth']"
          label="SSH Agent Auth"
          :desc="nativeProviderSchema?.config?.['ssh-agent-auth']?.description"
        />
      </template>
      <template v-else>
        <k-input
          label="Master"
          v-model="form.master"
          required />
        <k-input
          label="Worker"
          v-model="form.worker"
          required />
      </template>
    </div>
    <div>
      <k-alert v-for="(e, index) in errors" :key="index" type="error" :title="e"></k-alert>
    </div>
  </k-loading>
  </template>
  <template #footer>
        <k-button class="role-secondary" @click="visible = false">Cancel</k-button>
        <k-button class="role-danger" :loading="loading" @click="save">Save</k-button>
      </template>
</k-modal>
</template>
<script>
import {defineComponent, computed, inject, reactive, ref, toRef, watch} from 'vue'
import useCluster from '@/composables/useCluster.js'
import useProviders from '@/composables/useProviders.js'
import IpAddressPoolForm from '@/views/components/baseForm/IpAddressPoolForm.vue'
import BooleanForm from '@/views/components/baseForm/BooleanForm.vue'
import StringForm from '@/views/components/baseForm/StringForm.vue'
import { joinNode} from '@/api/cluster.js'
import { saveCreatingCluster} from '@/utils'
import {stringify} from '@/utils/error.js'

const formDefaultValue = {
  worker: '0',
  master: '0',
}
const nativeFormDefaultValue = {
  options: {
    'master-ips': '',
    'worker-ips': '',
  },
  'ssh-user': '',
  'ssh-port': '',
  'ssh-key-path': '',
  'ssh-key-passphrase': '',
  'ssh-cert-path': '',
  'ssh-password': '',
  'ssh-agent-auth': '',
}

export default defineComponent({
  props: {
    clusterId: {
      type: String,
      required: true
    },
    modelValue: {
      type: Boolean,
      default: false,
    }
  },
  emits: ['update:modelValue'],
  setup(props, {emit}) {
    const notificationStore = inject('notificationStore')
    const masterIps = ref(null)
    const workerIps = ref(null)

    const id = toRef(props, 'clusterId')
    const {loading: providersLoading, providers, error: loadProviderError} = useProviders()
    const {cluster, error: loadClusterError, loading: clusterLoading} = useCluster(id)
    const formErrors = ref([])
    const form = reactive({...formDefaultValue})
    const nativeForm = reactive({...nativeFormDefaultValue, options: { ...nativeFormDefaultValue.options }})
    watch(id, () => {
      Object.entries(formDefaultValue).forEach((e) => {
        form[e[0]] = e[1]
      })
      Object.entries(nativeFormDefaultValue).forEach((e) => {
        if (e[0] === 'options') {
           nativeForm[e[0]] = { ...nativeFormDefaultValue.options }
        } else {
          nativeForm[e[0]] = e[1]
        }
      })
    }, {
      immediate: true
    })
    const visible = computed({
      get() {
        return props.modelValue
      },
      set(v) {
        emit('update:modelValue', v)
      }
    })
    const nativeProviderSchema = computed(() => {
      return providers.value.find((p) => p.id === cluster.value?.provider)
    })
    const provider = computed(() => {
      return cluster.value?.provider
    })
    const desiredNodes = computed(() => {
      if (!cluster.value) {
        return {
            master: '0',
            worker: '0',
          }
      }
      return {
          master: `${parseInt(cluster.value.master, 10) + (parseInt(form.master, 10) || 0)}`,
          worker: `${parseInt(cluster.value.worker, 10) + (parseInt(form.worker, 10) || 0)}`,
        }
    })
    const errors = computed(() => {
      const errors = [];
      if (loadClusterError.value) {
        errors.push(loadClusterError.value)
      }
      if (loadProviderError.value) {
        errors.push(loadProviderError.value)
      }
      errors.push(...formErrors.value)
      return errors
    })

    const loading = computed(() => {
      return providersLoading.value || clusterLoading.value
    })

    const validate = () => {
      const errors = []
      formErrors.value = errors
      if (provider.value === 'native') {
        const formData = {...nativeForm, options: {
          'master-ips': masterIps.value.getForm().filter((v) => v).join(','),
          'worker-ips': workerIps.value.getForm().filter((v) => v).join(',')
        }}
        const masterIpsLen = formData.options['master-ips'].split(',').reduce((t, c) => {
          if (c.trim()) {
            t = t + 1
          }
          return t
        }, 0)
        const workerIpsLen = formData.options['worker-ips'].split(',').reduce((t, c) => {
          if (c.trim()) {
            t = t + 1
          }
          return t
        }, 0)

        if (masterIpsLen === 0 && workerIpsLen === 0) {
          errors.push('"Master IPs" and "Worker IPs" at least one');
        }
        
        return errors.length === 0
      }

      const formData = {...form}
      const numReg = /^[0-9]+$/

      if (!numReg.test(formData.worker)) {
        errors.push('"Worker" must a number');
      }
      if (!numReg.test(formData.master)) {
        errors.push('"Master" must be a number');
      }
      if (errors.length > 0) {
        return false
      }
      if (parseInt(formData.worker, 10) <= 0 && parseInt(formData.master, 10) <= 0) {
        errors.push('One of "Master" and "Worker" must be greater than 0');
      }
      return errors.length === 0
    }

    const save = async () => {
    if (!validate()) {
      return
    }
    try {
      let formData;
      if (provider.value === 'native') {
        formData = {...nativeForm, options: {
          'master-ips': masterIps.value.getForm().filter((v) => v).join(','),
          'worker-ips': workerIps.value.getForm().filter((v) => v).join(',')
        }}
      } else {
        formData = {...form}
      }
       
      const data = Object.assign({}, cluster.value, formData)
      await joinNode(data)
      saveCreatingCluster(data.id)
    } catch (err) {
      notificationStore.action.notify({
          type: 'error',
          title: 'Join Nodes Failed',
          content: stringify(err),
        })
    }
    visible.value = false
  }

    return {
      save,
      errors,
      loading,
      provider,
      form,
      nativeForm,
      desiredNodes,
      nativeProviderSchema,
      visible,
      masterIps,
      workerIps,
    }
  },
  components: {
    IpAddressPoolForm,
    BooleanForm,
    StringForm,
  }
})
</script>
