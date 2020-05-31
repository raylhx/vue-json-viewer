<template>
  <div class="app">
    <div class="main-container">
      <json-view :jsonData="testData"></json-view>
    </div>

  </div>
</template>

<script>

import JsonView from './components/vue-json-view'
import { mock } from './mock/test'
export default {
  components: {
    JsonView,
  },
  data () {
    return {
      testData: mock,
    }
  },

  mounted () {
    console.log('test', this.testData)
  },
}
</script>

<style lang="scss">
@import "./css/reset.css";
.app {
  padding: 50px;
}
.main-container {
  width: 900px;
}
</style>
