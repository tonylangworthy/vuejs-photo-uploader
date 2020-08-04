<template>
  <div id="app">
    <photo-uploader />
  </div>
</template>

<script>
import UIkit from 'uikit';
import Icons from 'uikit/dist/js/uikit-icons';
import PhotoUploader from './components/PhotoUploader.vue'

UIkit.use(Icons);

export default {
  name: 'App',
  components: {
    PhotoUploader
  }
}
</script>

<style lang="less">
@import "../node_modules/uikit/src/less/uikit.less";
</style>