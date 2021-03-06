<template>
  <div>
    <a-radio-group v-model="imported" @change="chooseHandle">
      <a-radio-button :value="false">未导入</a-radio-button>
      <a-radio-button :value="true">已导入</a-radio-button>
    </a-radio-group>
    <page-card-list
      :list="list"
      :card-fields="cardFields"
      :showPageer="false"
      :isRefreshed="false"
      :single-actions="singleActions" />
  </div>
</template>

<script>
import { arrToObjByKey, sizestr } from '@/utils/utils'
import WindowsMixin from '@/mixins/windows'
export default {
  name: 'ImageImport',
  mixins: [WindowsMixin],
  data () {
    return {
      imported: false,
      list: {
        limit: 20,
        total: 0,
        offset: 0,
        data: [],
        loading: false,
      },
      cardFields: {
        url: 'os',
        title: 'title',
        description: 'description',
        desc: 'desc',
      },
    }
  },
  computed: {
    singleActions () {
      return [
        {
          label: this.imported ? '已导入' : '导入',
          action: obj => {
            let data = {
              name: obj.name,
              copy_from: obj.url,
              disk_format: obj.format,
              is_public: true,
              is_standard: true,
              protected: true,
              properties: {
                os_type: obj.os_type,
                os_arch: obj.os_arch,
                os_description: obj.os_description,
                os_distribution: obj.os_distribution,
                os_version: obj.os_version,
              },
            }
            new this.$Manager('images', 'v1').create({ data })
              .then(() => {
                this.fetchData()
              })
          },
          meta: () => ({
            buttonType: 'primary',
            validate: !this.imported,
          }),
        },
      ]
    },
  },
  created () {
    this.fetchData()
  },
  methods: {
    fetchData () {
      this.list.loading = true
      new this.$Manager('imageutils/imagesinfo', 'v1').list().then(({ data }) => {
        this.list.loading = false
        let osDesc = (image) => {
          return `镜像格式: ${image.format || '-'}
                  镜像大小: ${image.disk || '-'}GiB
                  系统架构: ${image.arch || '-'}
                  文件大小: ${sizestr(image.size, 'B', 1024) || '-'},
                  更新日期: ${image.create_at || '-'}`
        }
        var publicImages = []
        for (let image of data) {
          if (!image.hasOwnProperty('distribution')) {
            console.log('warnning, image missing key distribution', image)
            continue
          }

          if (!image.hasOwnProperty('type')) {
            console.log('warnning, image missing key type', image)
            continue
          }

          let os = image.distribution.split(' ')[0].toLowerCase()
          let o = {}
          o['id'] = image.id
          o['name'] = image.name
          o['url'] = image.url
          o['format'] = image.format
          o['os_type'] = image.type
          o['os_arch'] = image.arch
          o['os_description'] = image.os_description
          o['os_distribution'] = image.distribution
          o['os_version'] = image.version
          o['class'] = `fo-${os} os-logo ${image['type'].toLowerCase()}`
          o['title'] = `${image.distribution} ${image.version}`
          o['desc'] = osDesc(image)
          o['imported'] = image.imported === 'true'
          o['os'] = os || image.type.toLowerCase()
          o['description'] = image.description
          publicImages.push(o)
        }
        publicImages = publicImages.filter((item) => { return item.imported === this.imported })
        publicImages.forEach(item => {
          item['os'] = require(`@/assets/images/system-icons/${item['os']}.svg`)
        })
        this.list.data = arrToObjByKey(publicImages, 'id')
      }).catch((e) => {
        this.list.loading = false
        throw e
      })
    },
    chooseHandle () {
      this.fetchData()
    },
  },
}
</script>
