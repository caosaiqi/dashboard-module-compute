<template>
  <page-list
    :list="list"
    :columns="columns"
    :group-actions="groupActions"
    :single-actions="singleActions" />
</template>

<script>
import {
  getEnabledTableColumn,
  getNameDescriptionTableColumn,
  getStatusTableColumn,
} from '@/utils/common/tableColumn'
import WindowsMixin from '@/mixins/windows'
import { sizestr } from '@/utils/utils'

export default {
  name: 'SkuList',
  mixins: [WindowsMixin],
  data () {
    return {
      list: this.$list.createList(this, {
        resource: 'serverskus',
        getParams: {
          details: true,
          with_meta: true,
        },
        filterOptions: {
          name: {
            label: '名称',
            filter: true,
            formatter: val => {
              return `name.contains(${val})`
            },
          },
        },
      }),
      columns: [
        getNameDescriptionTableColumn({
          vm: this,
          hideField: true,
          slotCallback: row => {
            return (
              <side-page-trigger onTrigger={ () => this.sidePageTriggerHandle(row.id, 'SkuSidePage') }>{ row.name }</side-page-trigger>
            )
          },
        }),
        {
          field: 'cpu_core_count',
          title: '虚拟CPU核数',
          width: 100,
          formatter: ({ cellValue }) => {
            return cellValue + '核'
          },
        },
        {
          field: 'memory_size_mb',
          title: '虚拟内存容量',
          width: 100,
          formatter: ({ cellValue }) => {
            return sizestr(cellValue, 'M', 1024)
          },
        },
        getStatusTableColumn({ statusModule: 'sku' }),
        {
          field: 'total_guest_count',
          title: '关联主机数量',
          width: 100,
        },
        getEnabledTableColumn(),
      ],
      groupActions: [
        {
          label: '新建',
          permission: 'skus_create',
          action: () => {
            this.createDialog('CreateSkuDialog', {
              title: '新建',
              list: this.list,
            })
          },
          meta: () => ({
            buttonType: 'primary',
          }),
        },
        {
          label: '更多',
          actions: () => {
            return [
              {
                label: '启用',
                permission: 'skus_update',
                action: () => {
                  this.list.batchPerformAction('enable', null)
                },
                meta: () => ({
                  validate: this.list.selectedItems.length,
                }),
              },
              {
                label: '禁用',
                permission: 'skus_update',
                action: () => {
                  this.list.batchPerformAction('disable', null)
                },
                meta: () => ({
                  validate: this.list.selectedItems.length,
                }),
              },
              {
                label: '删除',
                permission: 'skus_delete',
                action: () => {
                  this.createDialog('DeleteResDialog', {
                    data: this.list.selectedItems,
                    columns: this.columns,
                    title: '删除账号',
                    list: this.list,
                  })
                },
                meta: () => this.$getDeleteResult(this.list.selectedItems),
              },
            ]
          },
        },
      ],
      singleActions: [
        {
          label: '克隆',
          permission: 'skus_create',
          action: obj => {
            this.createDialog('CloneSkuUpdateDialog', {
              data: [obj],
              columns: this.columns,
              list: this.list,
            })
          },
          meta: obj => {
            let tooltip
            if (!obj.enabled) tooltip = '请先设置启用'
            return {
              validate: obj.enabled,
              tooltip,
            }
          },
        },
        {
          label: '更多',
          actions: obj => {
            return [
              {
                label: '启用',
                permission: 'skus_update',
                action: () => {
                  this.list.onManager('performAction', {
                    id: obj.id,
                    managerArgs: {
                      action: 'enable',
                    },
                  })
                },
                meta: () => ({
                  validate: !obj.enabled,
                }),
              },
              {
                label: '禁用',
                permission: 'skus_update',
                action: () => {
                  this.list.onManager('performAction', {
                    id: obj.id,
                    managerArgs: {
                      action: 'disable',
                    },
                  })
                },
                meta: () => ({
                  validate: obj.enabled,
                }),
              },
              {
                label: '删除',
                permission: 'skus_delete',
                action: () => {
                  this.createDialog('DeleteResDialog', {
                    data: [obj],
                    columns: this.columns,
                    title: '删除',
                    list: this.list,
                  })
                },
                meta: () => this.$getDeleteResult(obj),
              },
            ]
          },
        },
      ],
    }
  },
  created () {
    this.initSidePageTab('sku-detail')
    this.list.fetchData()
  },
}
</script>
