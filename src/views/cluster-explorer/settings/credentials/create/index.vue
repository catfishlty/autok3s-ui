<template>
  <div>
    <page-header>
      <template #title><router-link :to="{ name: 'ClusterExplorerSettingsCredentials' }">Credential: </router-link>Create</template>
    </page-header>
    <!-- fake fields are a workaround for chrome autofill getting the wrong fields -->
    <input style="display: none" type="text" />
    <input style="display: none" type="password" />
    <credential-form></credential-form>
  </div>
</template>
<script>
import {defineComponent} from 'vue'
import PageHeader from '@/views/components/PageHeader.vue'
import CredentialForm from '../components/CredentialCreateForm.vue'

export default defineComponent({
  name: 'CredentialCreate',
  components: {
    PageHeader,
    CredentialForm,
  }
})
</script>