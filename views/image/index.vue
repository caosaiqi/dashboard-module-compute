<template>
  <div>
    <page-header title="系统镜像" />
    <page-body>
      <image-list />
    </page-body>
  </div>
</template>

<script>
import ImageList from './components/List'

export default {
  name: 'ImageIndex',
  components: {
    ImageList,
  },
}
</script>
