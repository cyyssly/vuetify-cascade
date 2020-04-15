# vuetify-cascade
一个基于 vuetify 的级联选择框  
a cascade select component base on vuetify

### 1. 前置/Dependencies：vue、vuetify、lodash 

(1) 对组件的 `<template>` 部分稍加修改，也可以非常容易地支持其他组件库，但是意义不大，因为其他主流组件库已经自带了类似组件；  
A slight modification to the `<template>` part of the component can also easily support other component libraries, but it is not significant because other mainstream component libraries already have similar components.   
  
(2) 组件中只使用了 lodash 的 cloneDeep 函数，你也可以自行修改源码，使其不依赖 lodash。  
Only the `cloneDeep` function of `lodash` is used in the component, you can also modify the source code yourself to make it not depend on `lodash`.  

### 2. 特性/Features:
(1) 无限级数/Infinite cascade  
(2) 支持同步和异步数据/Support for synchronous and asynchronous data  
(3) 获取异步数据时支持 token 验证/Support token verification  
(4) 简短高效，只有90行代码/Short and efficient, only 90 lines of code  

![image](https://github.com/cyyssly/vue-vuetify-cascade/blob/master/1.JPG)

### 3. 使用方法/Instructions: 

#### 3.1. 复制 Cascade.vue 到项目目录下/Copy Cascade.vue to the project directory  
例如 src/components/ 下。 
such as src/components/.  

#### 3.2. 调用/Use： 

最常见的场景，是嵌套在一个下拉列表中： 
The most common scenario is to be nested in a drop-down list:   

```vue
  <v-menu v-model="item.menu" :close-on-content-click="false">
    <template v-slot:activator="{ on }">
      <v-text-field
        v-model="item.model"
        :label="item.label"
        readonly
        v-on="on"
      ></v-text-field>
    </template>
    <Cascade
      v-model="item.model"
      :Tabs="item.tabs"
      :Items="item.items"
      :AsyncMode="item.async"
      :ApiHref="item.api"
      @input="item.menu=!item.menu"
    ></Cascade>
  </v-menu>  
```
```js
<script>
import Cascade from '@/components/Cascade.vue'

export default {
  components: {
    Cascade
  },
  data() {
    return {
      item: {
        label: "省市区",
        model: [], // Array, values selected by the user at each level, for example: ['node1','node2','node3']
        tabs: ['level1', 'level2', 'level3'],
        items: [ {id:1, text:'item1'}, {id:2, text:'item2'} ], // options for level1
        menu: false,
        async: true,
        api: "/console/checkarea"
      }
    }
  }
</script>
```
  
### 4. 源码解析/Source code analysis  
Cascade 接受6个传入的参数，并通过 v-model 传出用户选择结果或接受预设值。  
Cascade accepts 6 incoming parameters, passes the user selection result or accepts preset values via the v-model  

#### 4.1. 入参/Incoming parameters

(1) height: String  on
组件高度：当选项数量较多时，或者独立使用而不是嵌套在下拉列表中时，用于限制组件的高度，默认为300px。  
Component height: When the number oof options is large, or when used independently instead of nested in a drop-down list, it is used to limit the height of the component. The default is 300px.  

(2) tabs: Array  
级别：格式为数组，例如：```['省', '市', '区']```，会生成一个三级的级联选择框。  
Level: such as ```['level1', 'level2', 'level3']```, Will generate a three-level cascade selection box.

(3) items: Array   
Items是一个复杂的嵌套对象数组，格式类似于：
Items is a complex array of objects:  
```js
  [
    [ {id:1, text:'item1'}, {id:2, text:'item2'} ],
    [ {id:1, text:'item1', pid:1}, {id:2, text:'item2', pid:1}, {id:3, text:'item3', pid:2} ],
    [ {id:1, text:'item1', pid:1}, {id:2, text:'item2', pid:2}, {id:3, text:'item3', pid:3} ]
  ]
```
其中第一层的数组为级联下拉选择框每一级的可选项。  
The first layer of the array is the option for each level of the cascade drop-down selection box.  
第二层的对象中 text 为该级选项显示内容，id 为选中值，pid 为父级节点 id。  
In the object of the second layer, text displays the content for the level option, id is the selected value, and pid is the parent node id.  

(4) asyncMode: Boolean    
如果数据量比较大，建议使用异步加载模式以改善性能。在异步加载模式下，组件初始化时 Items 属性只需要提供第一级选项的数据，  
If the amount of data is large, it is recommended to use asynchronous mode to improve performance. In asynchronous mode, the Items property only needs to provide data for the first level option when the component is initialized.  
其他层级数据将在选中第一级选项的具体节点后，通过访问 API 接口从后台异步获取。  
Other data will be acquired asynchronously by accessing the API interface after selecting the specific node of the first level option.  

(5) apiHref: String  
(6) token: String  
使用异步加载需要设置 AsyncMode 的值为 true，并提供获取后台数据的 API 地址。  
Using asynchronous loading requires setting the value of AsyncMode to true and providing an API address to get data.  
如有需要，您也可以传入 token ，供访问接口时使用。  
If necessary, you can also prop the token for use when accessing the interface.  
组件访问地址时会提供以下参数：1.level：数据层级，从0开始, 数值；2.pid：上级父节点的id, 字符串；3.token：令牌, 字符串   
The following parameters are provided when the component accesses the API: 1.level: data level, starting from 0, Number; 2.pid: id of the parent node, String; 3.token: token, String.   
API 需要实现的功能：根据传入的 pid 查询该节点的全部下级节点；  
The API needs to implement: query all subordinate nodes of the node according to the incoming param 'pid'.  
API 返回数据的格式要求与 Items 属性相同，是一个含有 id 和 text 属性的对象数组，不过对象中的 pid 可以忽略，例如：  
The format of the API return data is the same as the Items property, which is an array of objects with 'id' and 'text' properties, but the pid in the object can be ignored, for example:  
```js
  [ {id:1, text:'item1'}, {id:2, text:'item2'}, {id:3, text:'item3'} ]  
```
后端伪代码示例：
Backend pseudo code example: 
```node
async function(req, res) {
  const strSQL = "SELECT `code` as id,`name` as text FROM extarea where `parentcode`=?";
  const param = [req.body["pid"]];
  try {
    const results = await MysqlPool.dataQuery(strSQL, param);
    res.send(results);
  } catch (err) {
    err => res.status(500).send(err);
  }
}
```

#### 4.2. 返回值/Return value  

返回值为数组格式，分别是用户在每个级别选中的值，例如：
The return value is in array format, is the value selected by the user at each level, for example: 
```js
['node1','node2','node3']  
```
    
    

