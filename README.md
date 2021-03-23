# 这是一个支持超长列表加载的vue组件

+ 具体原理可以参考：[教程](https://appleking10.github.io/Appleking10.github.io/2020/11/27/Vue%E6%8F%92%E4%BB%B6-%E8%B6%85%E9%95%BF%E5%8D%95%E5%92%8C%E5%88%86%E7%BA%A7%E5%88%97%E8%A1%A8%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/)
## API
兼容两种列表，传入数组格式的数据，是单列表；传入Map对象的数据，是二级列表。
> 借用了Emement-ui的勾选框组件，并使用loadsh的节流和防抖，使用组件时需要安装两者

+ 参数

| 字段      | 描述      | 类型     |
| :---        |    :----:   |   :---: |
| datas      | 列表数据       | Array/ 必填   |
| label   | 数据显示名称        | String/选填，默认name   |
| nodekey   | 数据绑定名称        | String/选填，默认nodeKey   |
| itemH   | 列表一项的高度        | Number/选填，默认30px   |
| itemW   | 列表宽度        | Number/选填，若没传则为自适应宽度   |
| showNum   | 一屏显示条数        | Number/选填，默认5条   |
| remainNum   | 预加载条数        | Number/选填，默认5条   |
| isCheck   | 是否开启勾选框        | Boolean/选填，默认开启   |

+ 组件方法

| 方法名      | 描述      | 返回值     |
| :---        |    :----:   |   :---: |
| checked      | 勾选时的回调       | 目前所有被勾选的选项   |
| getSelectItem   | 获取当前所有勾选项        | 目前所有被勾选的选项   |

## 组件扩展应用

+ 下拉框
支持过滤功能，并且可以拼音模糊搜索

```
      import selectWrap from "../selectWrap.vue";
      <select-wrap :paramObj="paramObj"></select-wrap>
      
      paramObj = {
          data: [{
              name: "一级",
              nodeKey: "1",
              children: [{ name: '二级',nodekey: '1' },...]
          },...],
          nodeKey: "nodeKey",
          label: "name",
          id: 1,
      }
```

+ trnasfer框
支持过滤功能，并且可以拼音模糊搜索

```
      import transferWrap from "../transferPlugin.vue";
      <transfer-wrap :paramObj="paramObj2"></transfer-wrap>

      paramObj2 = {
          // data为map对象
          data[key] => {
              label: "name",
              nodeKey: "nodeKey", 
              children: [{
                  nodeKey: j,
                  name: `二级-${j}`,
                  parentKey: `一级-${i}`
              }],
          },
          nodeKey: "nodeKey",
          label: "name",
          id: 1,
      }
```
