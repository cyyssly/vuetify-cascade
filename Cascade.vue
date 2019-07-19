<template>
  <v-card :height="CHeight" style="min-width:300px;">
    <v-tabs
      v-model="CurrentTab"
      color="cyan"
      slider-color="yellow"
      grow
    >
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
    Height: String, // 组件高度：当选项数量较多时，用于限制组件的高度
    Tabs: Array, // 级别：格式为数组，例如：['省', '市', '区']，会生成一个三级的级联下拉选择框
    Items: Array,
    AsyncMode: Boolean,
    ApiHref: String
  },
  created () {
    this.CHeight = this.Height || 300
    this.CTabs = _.cloneDeep(this.Tabs)
    this.CItems = _.cloneDeep(this.Items)
  },
  methods: {
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
