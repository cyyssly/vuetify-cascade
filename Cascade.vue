<template>
  <v-card :height="CHeight" style="min-width:300px;">
    <v-tabs v-model="CurrentTab" color="cyan" slider-color="yellow" grow @change="changeTab(CurrentTab)">
      <!-- <v-tabs-slider color="yellow"></v-tabs-slider> -->
      <v-tab v-for="item in Tabs" :key="item">{{ item }}</v-tab>
    </v-tabs>
    <v-tabs-items v-model="CurrentTab">
      <v-tab-item v-for="(item, tindex) in Tabs" :key="item">
        <v-card flat>
          <v-list dense style="min-width:300px;" v-for="(item, iindex) in CItems" :key="item.index">
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
      CurrentTab: null, // 当前选中的级别
      CHeight: 300, // 默认组件高度
      CTabs: [],
      CItems: []
    }
  },
  props: {
    Height: String,
    Tabs: Array,
    Items: Array,
    AsyncMode: Boolean,
    ApiHref: String,
    token: String
  },
  created() {
    this.CHeight = this.Height || 300
    this.CTabs = cloneDeep(this.Tabs)
    this.CItems = cloneDeep(this.Items)
  },
  methods: {
    changeTab(tab) {
      // 切换 Tab 事件，如有需要，请重写该函数
      console.log(tab)
    },
    // 选中节点事件，如有需要，请重写该函数
    // 传入参数：1.item：当前选中的节点，对象，包含 id, text 和 pid；2.index：当前层级，数值，从0开始
    async selectNode(item, index) {
      // 是否异步加载模式
      if (this.AsyncMode) {
        const data = {}
        this.$set(data, "level", index)
        this.$set(data, "pid", item.id)
        try {
          const res = await this.$axios.post(this.ApiHref, data, { headers: { Authorization: this.token } })
          if (index < this.Items.length - 1) {
            // 非最后层级 not last level
            this.CItems[index + 1] = res.data
            this.CTabs[index] = item.text
            this.CurrentTab += 1
          } else {
            // 最后层级 last level
            this.CTabs[index] = item.text
            this.$emit("input", this.CTabs) // 返回值为数组格式，例如：['node1','node2','node3']
          }
        } catch (err) {
          this.$confirm("error", err.message || err)
        }
      } else {
        if (index < this.Items.length - 1) {
          // 非最后层级 not last level
          this.CItems[index + 1] = this.Items[index + 1].filter(node => node.pid === item.id)
          this.CTabs[index] = item.text
          this.CurrentTab += 1
        } else {
          // 最后层级 last level
          this.CTabs[index] = item.text
          this.$emit("input", this.CTabs) // 返回值为数组格式，例如：['node1','node2','node3']
        }
      }
    }
  }
}
</script>
