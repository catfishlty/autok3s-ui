<template>
  <!-- fake fields are a workaround for chrome autofill getting the wrong fields -->
  <input style="display: none" autocomplete="new-password" type="password" />
  <k-tabs tab-position="left" v-model="acitiveTab">
    <k-tab-pane label="Credential Options" name="credential">
      <form-group>
        <template #title>
          Credential Options
        </template>
        <template #default>
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-10px">
            <k-password-input
              v-model="form.options['access-key']"
              label="Access Key"
              :desc="desc.options['access-key']"
              :readonly="readonly">
            </k-password-input>
            <k-password-input
              v-model="form.options['secret-key']"
              label="Secret Key"
              :desc="desc.options['secret-key']"
              :readonly="readonly">
            </k-password-input>
          </div>
        </template>
      </form-group>
    </k-tab-pane>
    <k-tab-pane label="Instance Options" name="instance">
      <form-group>
        <template #title>
          Basic
        </template>
        <template #default>
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-10px">
            <string-form
              v-model.trim="form.options.region"
              label="Region"
              :desc="desc.options.region"
              :readonly="readonly"
            />
            <string-form
              v-model.trim="form.options.zone"
              label="Zone"
              :desc="desc.options.zone"
              :readonly="readonly"
            />
            <string-form
              v-model.trim="form.options['instance-type']"
              label="Instance Type"
              :desc="desc.options['instance-type']"
              :readonly="readonly"
            />
            <string-form
              v-model.trim="form.options['ami']"
              label="AMI"
              :desc="desc.options['ami']"
              :readonly="readonly"
            />
            <string-form
              v-model.trim="form.options['volume-type']"
              label="Volume Type"
              :desc="desc.options['volume-type']"
              :readonly="readonly"
            />
            <string-form
              v-model.trim="form.options['root-size']"
              label="Root Size"
              :desc="desc.options['root-size']"
              :readonly="readonly"
            />
            <boolean-form
              v-model="form.options['request-spot-instance']"
              label="Request Spot Instance"
              :desc="desc.options['request-spot-instance']"
              :readonly="readonly"
            />
            <string-form
              v-show="form.options['request-spot-instance']"
              v-model.trim="form.options['spot-price']"
              label="Spot Price"
              :desc="desc.options['spot-price']"
              :readonly="readonly"
            />
          </div>
        </template>
      </form-group>
      <hr class="section-divider">
      <form-group :closable="true">
        <template #title>
          Network
        </template>
        <template #default>
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-10px">
          <string-form
            v-model.trim="form.options['vpc-id']"
            label="VPC ID"
            :desc="desc.options['vpc-id']"
            :readonly="readonly"
          />
          <string-form
            v-model.trim="form.options['subnet-id']"
            label="Subnet ID"
            :desc="desc.options['subnet-id']"
            :readonly="readonly"
          />
          <string-form
            v-model.trim="form.options['security-group']"
            label="Security Group"
            :desc="desc.options['security-group']"
            :readonly="readonly"
          />
        </div>
        </template>
      </form-group>
      <hr class="section-divider">
      <form-group>
        <template #title>
          SSH Public
        </template>
        <template #subtitle>
          Params used to login to instance via ssh, e.g. key-pair, ssh user, ssh port
        </template>
        <template #default>
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-10px">
            <string-form
              v-model.trim="form.options['keypair-name']"
              label="Keypair Name"
              :desc="desc.options['keypair-name']"
              :readonly="readonly"
            />
            <string-form
              v-model.trim="form.config['ssh-user']"
              label="SSH User"
              :desc="desc.config['ssh-user']"
              :readonly="readonly"
            />
            <string-form
              v-model.trim="form.config['ssh-port']"
              label="SSH Port"
              :desc="desc.config['ssh-port']"
              :readonly="readonly"
            />
          </div>
        </template>
      </form-group>
      <hr class="section-divider">
      <form-group>
        <template #title>SSH Private</template>
        <template #subtitle>
          Params used to login to instance from user computer, e.g. ssh private key, ssh password, etc
        </template>
        <template #default>
          <ssh-private-form :form="form" :desc="desc" :readonly="readonly"></ssh-private-form>
        </template>
      </form-group>
      <hr class="section-divider">
      <form-group :closable="true">
        <template #title>Advance</template>
        <template #default>
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-10px">
            <cluster-tags-form
              ref="tags"
              v-model="form.options.tags"
              :desc="desc.options['tags']"
              :readonly="readonly"
              label="Tags"
              placeholder="e.g. foo=bar"
              action-label="Add Tag"></cluster-tags-form>
            </div>
        </template>
      </form-group>
    </k-tab-pane>
    <k-tab-pane label="K3s Options" name="k3s">
      <k3s-options-form
        :visible="acitiveTab === 'k3s'"
        :form="form"
        :desc="desc"
        :readonly="readonly">
      </k3s-options-form>
    </k-tab-pane>
    <k-tab-pane label="Additional Options" name="additional">
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-10px">
        <!-- <boolean-form
          v-model="form.config['ui']"
          label="UI"
          :desc="desc.config['enable'] || desc.config['ui']"
          :readonly="readonly"
        /> -->
        <k-select
          v-model="uiOptions"
          :desc="desc.config['enable']"
          label="UI"
          :disabled="readonly"
          placeholder="Disable"
          multiple
        >
          <k-option value="explorer" label="explorer"></k-option>
          <k-option value="dashboard" label="dashboard"></k-option>
        </k-select>
        <boolean-form
          v-model="form.options['cloud-controller-manager']"
          label="Cloud Controller Manager"
          :desc="desc.options['cloud-controller-manager']"
          :readonly="readonly"
        />
         <string-form
          v-show="form.options['cloud-controller-manager']"
          v-model.trim="form.options['iam-instance-profile-control']"
          label="IAM Instance Profile Control"
          :desc="desc.options['iam-instance-profile-control']"
          :readonly="readonly"
        />
         <string-form
          v-show="form.options['cloud-controller-manager']"
          v-model.trim="form.options['iam-instance-profile-worker']"
          label="IAM Instance Profile Worker"
          :desc="desc.options['iam-instance-profile-worker']"
          :readonly="readonly"
        />
      </div>
    </k-tab-pane>
  </k-tabs>
</template>
<script>
import { cloneDeep } from '@/utils'
import {defineComponent, ref, computed} from 'vue'
import BooleanForm from '../baseForm/BooleanForm.vue'
import StringForm from '../baseForm/StringForm.vue'
import K3sOptionsForm from '../baseForm/K3sOptionsForm.vue'
import SshPrivateForm from '../baseForm/SshPrivateForm.vue'
import ClusterTagsForm from '../baseForm/ArrayListForm.vue'
import FormGroup from '../baseForm/FormGroup.vue'
import useFormFromSchema from '../../composables/useFormFromSchema.js'
export default defineComponent({
  props: {
    schema: {
      type: Object,
      required: true,
    },
    readonly: {
      type: Boolean,
      default: false,
    },
  },
  setup(props) {
    const { form, desc }= useFormFromSchema(props.schema)
    const acitiveTab = ref('instance')
    const uiOptions = computed({
      get() {
        if (form.config.enable) {
          return form.config.enable
        }
        if (form.config.ui) {
          return ['dashboard']
        }
        return []
      },
      set(v) {
        form.config.enable = v
      }
    })
    const updateActiveTab = () => {
      if (!form.options['access-key'] || !form.options['secret-key']) {
        acitiveTab.value = 'credential'
        return
      }
      acitiveTab.value = 'instance'
    }
    updateActiveTab()

    const tags = ref(null)
    const getForm = () => {
      const f = cloneDeep(form)
      const values = tags.value.getForm()
      f.options.tags = values ? values.filter((v) => v) : values
      return f
    }
    return {
      form,
      desc,
      acitiveTab,
      getForm,
      tags,
      uiOptions,
    }
  },
  components: {
    BooleanForm,
    StringForm,
    K3sOptionsForm,
    SshPrivateForm,
    FormGroup,
    ClusterTagsForm,
  }
})
</script>
