<template>
  <v-card :height="CHeight" style="min-width:300px;">
    <v-tabs
      v-model="CurrentTab"
      color="cyan"
      slider-color="yellow"
      grow
      @change="changeTab(CurrentTab)"
    >
      <!-- <v-tabs-slider color="yellow"></v-tabs-slider> -->
      <v-tab v-for="item in Tabs" :key="item">{{ item }}</v-tab>
    </v-tabs>
    <v-tabs-items v-model="CurrentTab">
      <v-tab-item v-for="(item,tindex) in Tabs" :key="item">
        <v-card flat>
          <v-list dense style="min-width:300px;" v-for="(item,iindex) in CItems" :key="item.index">
            <template v-if="tindex===iindex">
              <v-list-tile v-for="it in item" :key="it.index" @click="selectNode(it,iindex)">
                <v-list-tile-content>{{it.text}}</v-list-tile-content>
              </v-list-tile>
            </template>
          </v-list>
        </v-card>
      </v-tab-item>
    </v-tabs-items>
  </v-card>
</template>
<script>
let _ = require('lodash/lang')

export default {
  data: () => ({
    CurrentTab: null, // 当前选中的级别
    CHeight: 300, // 默认组件高度
    CTabs: [],
    CItems: []
  }),
  props: {
    // 组件高度：当选项数量较多时，用于限制组件的高度
    Height: String,
    // 级别：格式为数组，例如：
    // ['省', '市', '区']，会生成一个三级的级联下拉选择框
    Tabs: Array,
    // items是一个复杂的嵌套对象数组，格式类似于：
    // [
    //   [ {id:1, text:'item1'}, {id:2, text:'item2'} ],
    //   [ {id:1, text:'item1', pid:1}, {id:2, text:'item2', pid:1}, {id:3, text:'item3', pid:2} ],
    //   [ {id:1, text:'item1', pid:1}, {id:2, text:'item2', pid:2}, {id:3, text:'item3', pid:3} ]
    // ]
    // 其中第一层的数组为级联下拉选择框每一级的可选项
    // 第二层的对象中text为该级选项显示内容，id为选中值，pid为父级节点id
    Items: Array,
    // 如果数据量比较大，建议使用异步加载模式以改善性能。在异步加载模式下，组件初始化时 Items 属性只需要提供第一级选项的数据，
    // 其他层级数据将在选中第一级选项的具体节点后，通过访问 API 接口从后台异步获取。
    // 使用异步加载需要设置 AsyncMode 的值为 true，并提供获取后台数据的 API 地址
    // 组件访问地址时会提供以下参数：1.level：数据层级，从0开始, 数值；2.pid：上级父节点的id, 字符串
    // API 返回数据的格式要求与 Items 属性相同，是一个对象数组，不过对象中的 pid 可以忽略，例如：
    // [ {id:1, text:'item1'}, {id:2, text:'item2'}, {id:3, text:'item3'} ]
    AsyncMode: Boolean,
    ApiHref: String
  },
  created () {
    this.CHeight = this.Height || 300
    this.CTabs = _.cloneDeep(this.Tabs)
    this.CItems = _.cloneDeep(this.Items)
  },
  methods: {
    changeTab (tab) { // 切换 Tab 事件，如有需要，请重写该函数
      console.log(tab)
    },
    // 选中节点事件，如有需要，请重写该函数
    // 返回参数：1.item：当前选中的节点，对象，包含 id, text 和 pid；2.index：当前层级，数值，从0开始
    async selectNode (item, index) {
      // 是否异步加载模式
      if (this.AsyncMode) {
        let data = {}
        this.$set(data, "level", index)
        this.$set(data, "pid", item.id)
        try {
          const res = await this.$axios.post(this.ApiHref, data)
          if (index < this.Items.length - 1) {
            this.CItems[index + 1] = res.data
            this.CTabs[index] = item.text
            this.CurrentTab += 1
          } else {
            this.CTabs[index] = item.text
            this.$emit('input', this.CTabs)
          }
        } catch (err) {
          const conStr = { type: 'error', msg: err.message || err, btnyes: { color: 'error' }, btnno: { visible: false } }
          this.$confirm(conStr)
        }
      } else {
        if (index < this.Items.length - 1) {
          this.CItems[index + 1] = this.Items[index + 1].filter(node => node.pid === item.id)
          this.CTabs[index] = item.text
          this.CurrentTab += 1
        } else {
          this.CTabs[index] = item.text
          // 返回值为数组格式，例如：['node1','node2','node3']
          this.$emit('input', this.CTabs)
        }
      }
    }
  }
}
</script>
