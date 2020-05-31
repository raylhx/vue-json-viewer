<template>
  <div class="json-view-container">
    <div class="json-view">
      <span
        v-if="length"
        class="angle"
        @click="toggleClose">
        <svg
          v-if="innerclosed"
          :fill="'#c6c6c6'"
          width="1em"
          height="1em"
          viewBox="0 0 1792 1792"
          style="vertical-align: middle; color: rgb(42, 161, 152); height: 1em; width: 1em;">
          <path d="M1344 800v64q0 14-9 23t-23 9h-352v352q0 14-9 23t-23 9h-64q-14 0-23-9t-9-23v-352h-352q-14 0-23-9t-9-23v-64q0-14 9-23t23-9h352v-352q0-14 9-23t23-9h64q14 0 23 9t9 23v352h352q14 0 23 9t9 23zm128 448v-832q0-66-47-113t-113-47h-832q-66 0-113 47t-47 113v832q0 66 47 113t113 47h832q66 0 113-47t47-113zm128-832v832q0 119-84.5 203.5t-203.5 84.5h-832q-119 0-203.5-84.5t-84.5-203.5v-832q0-119 84.5-203.5t203.5-84.5h832q119 0 203.5 84.5t84.5 203.5z">
          </path>
        </svg>
        <svg
          v-if="!innerclosed"
          :fill="'#c6c6c6'"
          width="1em"
          height="1em"
          viewBox="0 0 1792 1792"
          style="vertical-align: middle; color: rgb(88, 110, 117); height: 1em; width: 1em;">
          <path d="M1344 800v64q0 14-9 23t-23 9h-832q-14 0-23-9t-9-23v-64q0-14 9-23t23-9h832q14 0 23 9t9 23zm128 448v-832q0-66-47-113t-113-47h-832q-66 0-113 47t-47 113v832q0 66 47 113t113 47h832q66 0 113-47t47-113zm128-832v832q0 119-84.5 203.5t-203.5 84.5h-832q-119 0-203.5-84.5t-84.5-203.5v-832q0-119 84.5-203.5t203.5-84.5h832q119 0 203.5 84.5t84.5 203.5z">
          </path>
        </svg>
      </span>
      <div class="content-wrapper">
        <p class="first-line">
          <span v-if="jsonkey" class="json-key">"{{ jsonkey }}":</span>
          <span v-if="length">
            {{ prefix }}{{ innerclosed ? `...${subfix}` : '' }}
            <span v-if="innerclosed" class="json-note">{{ `${length}items` }}</span>
          </span>
          <span v-if="!length">{{ `${isArray ? '[]' : '{}'}${isLast ? '' : ','}` }}</span>
        </p>
        <div v-if="!innerclosed && length" class="json-body">
          <template v-for="(item, index) in jsonItems">
            <json-view
              v-if="item.isJSON"
              :key="index"
              :closed="isClose()"
              :isLast="index === jsonItems - 1"
              :jsonkey="item.key"
              :deep="deep"
              :currentDeep="templateDeep"
              :jsonData="item.value"></json-view>
            <p
              v-else
              :key="index"
              class="json-item">
              <span class="json-key">{{ isArray ? '' :`"${item.key}"` }}</span>
              <span class="json-value">{{ `${isStringType ? '"':''}${item.value}${isStringType ? '"':''}${index === jsonItems.length - 1 ? '' : ','}` }}</span>
            </p>
          </template>
          <span v-if="!innerclosed" class="base-line"></span>
        </div>
        <p v-if="!innerclosed" class="last-line"><span>{{ subfix }}</span></p>
      </div>
    </div>
  </div>
</template>

<script>
// util
function toRawType (value) {
  return Object.prototype.toString.call(value).slice(8, -1).toLowerCase()
}
function isObjectOrArray (value) {
  return ['array', 'object'].includes(toRawType(value))
}
function isArray (value) { return toRawType(value) === 'array' }

function isString (value) { return toRawType(value) === 'string' }

// todo 这个判空的应该还有更优解
function isEmptyArrayOrObject (value) {
  return [[], {}].map(item => JSON.stringify(item)).includes(JSON.stringify(value))
}
// vue
export default {
  name: 'JsonView',
  props: {
    jsonData: {
      type: [Object, Array],
      default: function () { return {} },
    },
    jsonkey: {
      type: String,
      default: '',
    },
    isLast: {
      type: Boolean,
      default: false,
    },
    closed: {
      type: Boolean,
      default: false,
    },
    deep: {
      type: Number,
      default: 4,
    },
    currentDeep: { // 记录递归深处
      type: Number,
      default: 1,
    },
  },
  data () {
    return {
      innerclosed: this.closed,
      templateDeep: this.currentDeep + 1,
      visible: false,
    }
  },
  computed: {
    // 想把这个isArray拆掉
    isArray () {
      return isArray(this.jsonData)
    },
    length () {
      return this.isArray ? this.jsonData.length : Object.keys(this.jsonData).length
    },
    jsonItems () {
      let data = this.jsonData
      if (this.isArray) {
        return data.map(item => {
          return {
            value: item,
            isJSON: isObjectOrArray(item),
            key: '',
          }
        })
      }
      return Object.keys(data).map(key => {
        return {
          value: data[key],
          isJSON: isObjectOrArray(data[key]),
          key,
        }
      })
    },
    prefix () {
      return this.isArray ? '[' : '{'
    },
    subfix () {
      const data = this.jsonData
      if (isEmptyArrayOrObject(data)) {
        return ''
      }
      return `${this.isArray ? ']' : '}'}${this.isLast ? '' : ','}`
    },
  },
  watch: {
    closed () {
      this.innerclosed = this.closed
    },
  },
  mounted () {
    this.init()
  },
  methods: {
    init () {
      setTimeout(() => {
        this.visible = true
      }, 0)
    },
    isStringType (value) {
      isString(value)
    },
    toggleClose () {
      if (this.length === 0) return
      this.innerclosed = !this.innerclosed
    },
    // 控制层级 小于既定deep深度的不再展开，为 close
    isClose () {
      return this.templateDeep > this.deep
    },
  },
}
</script>
<style lang="scss">
@import "~@/css/helper";
.json-view-container {
  background: #1e1e1e;
}
.json-view-container .json-view {
  position: relative;
  width: 100%;
  height: 100%;
  padding-left: 32px;
  box-sizing: border-box;
  white-space: nowrap;
  font-size: 14px;
  font-family: Menlo, Consolas, "Courier New", Courier, FreeMono, monospace !important;
}
.json-view .angle {
  position: absolute;
  top: 0;
  left: 14px;
  display: block;
  width: 20px;
  height: 20px;
  text-align: center;
}
.json-view .first-line,
.json-view .last-line {
  color: #d4d4d4;
  line-height: 24px;
}

.json-view .json-item {
  padding-left: 32px;
  color: #d4d4d4;
  line-height: 24px;
}
.json-view .json-note {
  color: #909399;
  font-style: italic;
}
.json-view .json-key {
  color: #a9dbfb;
}
.json-view .json-value {
  color: #c6937c;
}
.json-body {
  position: relative;
  margin: 0;
  padding: 0;
}
.json-body .base-line {
  position: absolute;
  height: 100%;
  top: 0;
  left: 2px;
  border-left: 1px solid #404040;
}
</style>
