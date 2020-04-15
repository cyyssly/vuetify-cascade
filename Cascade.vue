<template>
  <v-card :height="cHeight" style="min-width:300px;">
    <v-tabs v-model="currentTab" color="cyan" slider-color="yellow" grow @change="changeTab(currentTab)">
      <!-- <v-tabs-slider color="yellow"></v-tabs-slider> -->
      <v-tab v-for="item in tabs" :key="item">{{ item }}</v-tab>
    </v-tabs>
    <v-tabs-items v-model="currentTab">
      <v-tab-item v-for="(item, tindex) in tabs" :key="item">
        <v-card flat>
          <v-list dense style="min-width:300px;" v-for="(item, iindex) in cItems" :key="item.index">
            <template v-if="tindex === iindex">
              <v-list-item v-for="it in item" :key="it.index" @click="selectNode(it, iindex)">
                <v-list-item-content>{{ it.text }}</v-list-item-content>
              </v-list-item>
            </template>
          </v-list>
        </v-card>
      </v-tab-item>
    </v-tabs-items>
  </v-card>
</template>
<script>
import cloneDeep from "lodash/cloneDeep"

export default {
  data() {
    return {
      currentTab: null, // 当前选中的级别
      cHeight: 300, // 默认组件高度
      cTabs: [],
      cItems: []
    }
  },
  props: {
    height: String,
    tabs: Array,
    items: Array,
    // [ {id:1, text:'item1'}, {id:2, text:'item2'}, {id:3, text:'item3'} ]
    asyncMode: Boolean,
    apiHref: String,
    token: String
  },
  created() {
    this.cHeight = this.height || 300
    this.cTabs = cloneDeep(this.tabs)
    this.cItems = cloneDeep(this.items)
  },
  methods: {
    changeTab(tab) {
      // 切换 Tab 事件，如有需要，请重写该函数
      console.log(tab)
    },
    // 选中节点事件，如有需要，请重写该函数
    // 传入参数：1.item：当前选中的节点，对象，包含 id, text 和 pid；2.index：当前层级，数值，从0开始
    async selectNode(item, index) {
      if (this.asyncMode) {
        const data = {}
        this.$set(data, "level", index)
        this.$set(data, "pid", item.id)
        try {
          const res = await this.$axios.post(this.apiHref, data, { headers: { Authorization: this.token } })
          if (index < this.items.length - 1) {
            // 非最后层级 not last level
            this.cItems[index + 1] = res.data
            this.cTabs[index] = item.text
            this.currentTab += 1
          } else {
            // 最后层级 last level
            this.cTabs[index] = item.text
            this.$emit("input", this.cTabs) // 返回值为数组格式，例如：['node1','node2','node3']
          }
        } catch (err) {
          this.$confirm("error", err.message || err)
        }
      } else {
        if (index < this.items.length - 1) {
          // 非最后层级 not last level
          this.cItems[index + 1] = this.items[index + 1].filter(node => node.pid === item.id)
          this.cTabs[index] = item.text
          this.currentTab += 1
        } else {
          // 最后层级 last level
          this.cTabs[index] = item.text
          this.$emit("input", this.cTabs) // 返回值为数组格式，例如：['node1','node2','node3']
        }
      }
    }
  }
}
</script>
