<template>
  <base-dialog @cancel="cancelDialog">
    <div slot="header">挂载</div>
    <div slot="body">
      <dialog-selected-tips :count="params.data.length" action="挂载" />
      <vxe-grid class="mb-2" :data="params.data" :columns="params.columns.slice(0, 3)" />
      <a-form
        :form="form.fc">
        <a-form-item label="云服务器" v-bind="formItemLayout">
          <a-select v-decorator="decorators.server">
            <a-select-option v-for="item in serversOpts" :key="item.id">
              {{item.name}}
            </a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </div>
    <div slot="footer">
      <a-button type="primary" @click="handleConfirm" :loading="loading">{{ $t('dialog.ok') }}</a-button>
      <a-button @click="cancelDialog">{{ $t('dialog.cancel') }}</a-button>
    </div>
  </base-dialog>
</template>

<script>
import DialogMixin from '@/mixins/dialog'
import WindowsMixin from '@/mixins/windows'

export default {
  name: 'MountUpdateDialog',
  mixins: [DialogMixin, WindowsMixin],
  data () {
    return {
      loading: false,
      form: {
        fc: this.$form.createForm(this),
      },
      decorators: {
        server: [
          'server',
          {
            rules: [
              { required: true, message: '请选择云服务器' },
            ],
          },
        ],
      },
      formItemLayout: {
        wrapperCol: {
          span: 21,
        },
        labelCol: {
          span: 3,
        },
      },
      serversOpts: [],
    }
  },
  created () {
    let params = {
      details: false,
      disk: this.params.data[0].id,
    }
    new this.$Manager('servers').list({ params })
      .then((res) => {
        this.serversOpts = res.data.data
      })
  },
  methods: {
    validateForm () {
      return new Promise((resolve, reject) => {
        this.form.fc.validateFields((err, values) => {
          if (!err) {
            resolve(values)
          } else {
            reject(err)
          }
        })
      })
    },
    doUpdate (data) {
      return new this.$Manager('servers').batchPerformAction({
        action: 'attachdisk',
        ids: data.server,
        data: {
          disk_id: this.params.data[0].id,
        },
      })
    },
    async handleConfirm () {
      this.loading = true
      try {
        const values = await this.validateForm()
        this.loading = true
        await this.doUpdate(values)
        this.loading = false
        this.params.list.refresh()
        this.cancelDialog()
      } catch (error) {
        this.loading = false
      }
    },
  },
}
</script>
