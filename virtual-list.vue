<!--
  功能：虚拟列表加载插件
  作者：jinyini
  时间：2020年04月27日 15:01:54
  版本：v2.0
  应用名称：
-->
<template>
  <!-- 主容器  -->
  <div
    class="viewport"
    ref="view"
    @scroll="scrolFn"
    :style="{ maxWidth: setMaxWidth }"
  >
    <!-- 虚拟列表，用paddingTop来做偏移量 -->
    <div
      class="virtual-list"
      ref="virtualList"
      :style="{ paddingTop: `${offset}px`, lineHeight: `${itemH}px` }"
    >
      <div v-show="datas.length > 0 || datas.size > 0">
        <div
          class="check-box"
          v-bind:class="{ 'el-icon-check': checkedSet.has('checkAll') }"
          @click="selectAll"
        ></div>
        <span>全选</span>
      </div>
      <div
        v-for="(item, index) in virtualList"
        class="item"
        :key="item[0] + '-' + index"
        ref="item"
      >
        <!-- 单级列表 -->
        <div
          class="single-list"
          v-if="Array.isArray(datas)"
          :style="{ height: `${itemH}px` }"
        >
          <div
            v-if="isCheck"
            class="check-box"
            v-bind:class="{
              'el-icon-check': checkedSet.has(
                item[label] + '-' + item[nodekey]
              ),
            }"
            @click="selectItem(item, item['parentKey'])"
          ></div>
          <!-- 插槽：放置每一项列表，组件通过slot-scope="{item}"接收 -->
          <slot name="singleList" :item="item">{{ item.name }}</slot>
        </div>
        <!-- 多级列表 -->
        <div
          class="main-list"
          :style="{ height: `${itemH}px` }"
          @click="toggleFold(index, item)"
          v-else
        >
          <span style="display: inline-block; width: 12px">
            <span
              class="el-icon-caret-right"
              :style="{
                transform: foldFlag[index] ? 'rotate(90deg)' : 'rotate(0deg)',
              }"
              v-show="datas && datas.get(item[0])['children'].length > 0"
            ></span>
            <!-- <div>{{datas.get(item[0])['children'].length}}</div> -->
          </span>

          <div
            class="check-box"
            v-bind:class="{ 'el-icon-check': checkedSet.has(item[0]) }"
            @click.stop="selectAllChildren(item[0])"
          ></div>
          <slot name="main" :main="item[0]">{{ item[0] }}</slot>
        </div>

        <ul
          class="fold_tree"
          style="padding-left: 30px; list-style: none; margin: 0"
          v-if="
            !Array.isArray(datas) &&
            foldFlag[index] &&
            item[1]['children'].length > 0
          "
        >
          <li
            v-for="road in item[1]['children'].slice()"
            style="white-space: nowrap"
            :key="road[label] + '-' + road[nodekey]"
          >
            <div
              class="check-box"
              v-bind:class="{
                'el-icon-check': checkedSet.has(
                  road[label] + '-' + road[nodekey]
                ),
              }"
              @click="selectItem(road, item[0])"
            ></div>

            <slot name="sub" :sub="road">{{ road[label] }}</slot>
          </li>
        </ul>
      </div>
      <div v-if="datas.length == 0 || datas.size == 0">暂无数据</div>
    </div>
    <div class="scrollToTop el-icon-back" v-if="backToTop" @click="scrollToTop">
      <slot name="scrollToTop"></slot>
    </div>
  </div>
</template>
<script>
import _ from "lodash";
import throttle from "lodash/throttle";
// import _ from "lodash";

export default {
  props: {
    itemH: { type: Number, default: 30 }, //一行多高,默认30px
    itemW: null, //固定宽度，若没传则为不定宽
    showNum: { type: Number, default: 5 }, //要展示多少条，默认5条
    datas: null, //可以接收数组和map，数组=>单列表，map=>二级列表[必填]
    remainNum: { type: Number, default: 5 }, //预留加载多少条，
    isCheck: { type: Boolean, default: true }, //是否开启选择框
    backToTop: { type: Boolean, default: false }, //是否开启回到顶部按钮
    label: { type: String, default: "name" }, //数据显示名称
    nodekey: { type: String, default: "nodeKey" }, //数据绑定值
  },
  data() {
    return {
      start: 0, //列表开始
      end: this.showNum, //默认显示
      offset: 0, //虚拟列表的偏移量
      isArray: true, //判断是不是数组类型，false为map类型
      foldFlag: [], //展开标志
      unFoldIndex: null, //当前展开列表id
      checkedFlag: 0, //一级菜单选择项
      checkedSet: null, //保存选中项的key(label+"-"+nodekey),
    };
  },
  created() {
    //滚动节流
    this.scrolFn = throttle(this.handleScroll, 100, { leading: false });
    // 判断是否是单列表
    this.isArray = Array.isArray(this.datas);

    // 备份原始数据
    if (this.isArray) {
      this.dataRecord = this.datas.slice(0);
    } else {
      this.dataRecord = new Map(this.datas);
    }
    // 勾选集
    this.checkedSet = new Set();
  },
  mounted() {
    this.$refs.view.style.height = this.itemH * (this.showNum + 1) + "px";

    if (this.isArray) {
      //初始化滚动条的高度和主容器的高度
      this.$refs.virtualList.style.height =
        this.itemH * (1 + this.datas.length) + "px";
      //初始化第一屏列表
      this.end = this.start + this.showNum;
    } else {
      // 二级列表
      let scrollH = this.itemH * (this.datas.size + 1);
      this.$refs.virtualList.style.height = scrollH + "px";
      this.end =
        this.start + this.showNum > this.datas.size
          ? this.showNum
          : this.datas.size;
    }
  },
  computed: {
    //虚拟列表的切割
    virtualList() {
      if (this.datas) {
        if (Array.isArray(this.datas)) {
          return this.datas.slice(
            this.start - this.prevCount,
            this.end + this.nextCount
          );
        } else {
          if (this.unFoldIndex === null) {
            //若当前没有展开
            return this.datas;
          } else {
            //若当前展开了，找到展开的一级菜单，切割当前列表
            //深拷贝对性能不太好
            // let data = _.cloneDeep(this.datas);
            let data = new Map();
            //优化后写法
            this.datas.forEach((road, name) => {
              let obj = { children: [], id: road.id };
              obj[this.label] = road[this.label];
              obj[this.nodekey] = road[this.nodekey];
              data.set(name, obj);
            });

            let key = [...this.datas][this.unFoldIndex][0];
            let roadArr = [...this.datas][this.unFoldIndex][1][
              "children"
            ].slice(this.start - this.prevCount, this.end + this.nextCount);
            let newObj = data.get(key);
            newObj["children"] = roadArr;
            data.set(key, newObj);

            return data;
          }
        }
      }
      return [];
    },
    //在虚拟列表前后占一品位,返回位置
    prevCount() {
      //若当前开始位置大于展示条数，说明需要占位
      return Math.min(this.start, this.remainNum);
    },
    nextCount() {
      //若到达列表末尾，则返回剩下条数
      if (Array.isArray(this.datas)) {
        return Math.min(this.datas.length - this.end, this.remainNum);
      } else {
        //若当前没有展开
        if (this.unFoldIndex === null) {
          return Math.min(this.datas.size - this.end, this.remainNum);
        } else {
          //若展开，找到当前展开二级菜单所有数据
          let data = [...this.datas][this.unFoldIndex][1]["children"];
          return Math.min(data.length - this.end, this.remainNum);
        }
      }
    },
    setMaxWidth() {
      // 宽度自适应
      if (this.itemW) {
        return parseInt(this.itemW) + "px";
      } else {
        return "unset";
      }
    },
  },
  watch: {
    datas(val, oval) {
      let scrollTop = this.$refs.view.scrollTop;
      this.$refs.view.scrollTop = 0;
      this.offset = 0;
      if (Array.isArray(val)) {
        this.$refs.realWidth.style.height =
          (1 + val.length) * this.itemH + "px";
      } else {
        let height = (1 + val.size) * this.itemH;
        if (this.unFoldIndex != null) {
          height +=
            [...val][this.unFoldIndex][1]["children"].length * this.itemH;
        }
        this.$refs.virtualList.style.height = height + "px";
        //transfer的情况下
        if (!Array.isArray(oval)) {
          this.dataRecord = new Map(val);
          this.$refs.view.scrollTop = scrollTop;
        }
      }
    },
    checkedSet(val, oval) {
      this.checkedFlag = val.size;
    },
    checkedFlag(val, oval) {
      this.$emit("checked", this.checkedSet);
    },
  },
  methods: {
    handleScroll() {
      let scrollTop = this.$refs.view.scrollTop;
      if (Array.isArray(this.datas)) {
        this.start = Math.floor(scrollTop / this.itemH); //向下取整,滑过多少个了
        this.end = this.start + this.showNum;
        //如果有预留渲染，向上移动
        this.offset = this.start * this.itemH - this.prevCount * this.itemH;
      } else {
        //如果有展开，才会去改变展示数据
        if (this.unFoldIndex != null) {
          //如果滑动距离超过展开上面一级菜单的长度
          if (scrollTop > (this.unFoldIndex + 2) * this.itemH) {
            this.start = Math.floor(
              (scrollTop - (this.unFoldIndex + 1) * this.itemH) / this.itemH
            );
            this.offset = this.start * this.itemH - this.prevCount * this.itemH;
          } else {
            this.start = 0;
            this.offset = 0;
          }
          this.end = this.start + this.showNum;
        }
      }
    },
    //只能展开一个二级菜单
    toggleFold(index, item) {
      let scrollH;

      if (this.unFoldIndex === index) {
        //若当前点击等于自己，则收起
        this.$set(this.foldFlag, index, !this.foldFlag[index]);
        this.unFoldIndex = null;
        scrollH = this.itemH * (this.datas.size + 1);
      } else if (this.unFoldIndex == null) {
        //若是当前没有展开，则展开当前点击
        this.$set(this.foldFlag, index, true);

        scrollH =
          this.itemH *
          (this.datas.size + 1 + this.datas.get(item[0])["children"].length);
        this.unFoldIndex = index;
        //算上当前页面已有的一级列表高度
      } else {
        //若点击时已有别的列表展开，便收起其它列表，展开当前列表
        this.$set(this.foldFlag, this.unFoldIndex, false);
        this.$set(this.foldFlag, index, true);

        scrollH =
          this.itemH *
          (this.datas.size + 1 + this.datas.get(item[0])["children"].length);

        this.unFoldIndex = index;
      }
      //不管展开或者收起，都将指针初始化
      this.start = 0; //控制当前列表的指针
      this.end = this.start + this.showNum;

      this.$refs.virtualList.style.height = scrollH + "px";
    },
    scrollToTop() {
      if (this.unFoldIndex != null) {
        //展开二级时滚到当前一级菜单位置
        this.$refs.view.scrollTop = this.unFoldIndex * this.itemH;
      } else {
        //否则回到顶部
        this.$refs.view.scrollTop = 0;
      }
    },
    selectAll() {
      //目前全选，则全不选
      if (this.checkedSet.has("checkAll")) {
        this.checkedSet.clear();
      } else {
        if (this.isArray) {
          this.dataRecord.forEach((item) => {
            let key = item[this.label] + "-" + item[this.nodekey];
            this.checkedSet.add(key);
          });
        } else {
          //全部添加
          this.dataRecord.forEach((val, key) => {
            this.checkedSet.add(key);
            val["children"].forEach((children) => {
              let childrenKey =
                children[this.label] + "-" + children[this.nodekey];
              this.checkedSet.add(childrenKey);
            });
          });
        }
        this.checkedSet.add("checkAll");
        this.checkedFlag = this.checkedSet.size;
      }
      this.$forceUpdate();
    },
    selectAllChildren(key) {
      // 一级菜单的全选或者全部不选

      let list = this.dataRecord.get(key)["children"];
      if (this.checkedSet.has(key)) {
        //如果已经选中了,取消全选，把子集全部删掉
        this.checkedSet.delete(key);
        for (let i = 0, len = list.length; i < len; i++) {
          let childrenKey = list[i][this.label] + "-" + list[i][this.nodekey];

          this.checkedSet.delete(childrenKey);
          this.checkedSet.delete("checkAll");
        }
      } else {
        //全选，把子级全部加上,判断是不是一级全选
        this.checkedSet.add(key);
        for (let i = 0, len = list.length; i < len; i++) {
          let childrenKey = list[i][this.label] + "-" + list[i][this.nodekey];
          this.checkedSet.add(childrenKey);
        }
        let selectAllFlag = true;
        this.dataRecord.forEach((val, parentKey) => {
          if (!this.checkedSet.has(parentKey)) {
            selectAllFlag = false;
          }
        });
        if (selectAllFlag) {
          this.checkedSet.add("checkAll");
        } else {
          this.checkedSet.delete("checkAll");
        }
      }
      this.$nextTick(() => {
        this.$set(this.$data, "checkedSet", this.checkedSet);
      });
      this.checkedFlag = this.checkedSet.size;

      this.$forceUpdate();
    },
    //单独选择一项
    selectItem(item, parentKey) {
      let key = item[this.label] + "-" + item[this.nodekey];

      if (this.isArray) {
        if (this.checkedSet.has(key)) {
          this.checkedSet.delete(key);
          this.checkedSet.delete("checkAll");
        } else {
          this.checkedSet.add(key);
          if (this.checkedSet.size == this.dataRecord.length) {
            this.checkedSet.add("checkAll");
          }
        }
      } else {
        // this.$nextTick(() => {
        let currentList = this.dataRecord.get(parentKey)["children"];
        if (this.checkedSet.has(key)) {
          //当前有勾选，取消勾选，二级和一级肯定取消勾选
          this.checkedSet.delete(key);
          this.checkedSet.delete(parentKey);
          this.checkedSet.delete("checkAll");
        } else {
          //当前没有勾选，勾选上，并判断所有二级有勾选？
          this.checkedSet.add(key);
          let check = currentList.every((currentItem) => {
            //所有二级都有勾选
            let currentKey =
              currentItem[this.label] + "-" + currentItem[this.nodekey];
            return this.checkedSet.has(currentKey);
          });
          if (check) {
            this.checkedSet.add(parentKey);
          } else {
            this.checkedSet.delete(parentKey);
          }
          let selectAllFlag = true;
          //判断一级是否全部勾选
          this.dataRecord.forEach((val, parentsKey) => {
            if (!this.checkedSet.has(parentsKey)) {
              selectAllFlag = false;
            }
          });
          if (selectAllFlag) {
            this.checkedSet.add("checkAll");
          } else {
            this.checkedSet.delete("checkAll");
          }
        }
      }
      this.$nextTick(() => {
        this.$set(this.$data, "checkedSet", this.checkedSet);
      });
      this.checkedFlag = this.checkedSet.size;
      this.$forceUpdate();
    },
    getSelectItem() {
      let results = [];
      if (this.isArray) {
        if (this.checkedSet.has("checkAll")) {
          this.checkedSet.delete("checkAll");
          results = Array.from(this.checkedSet);
          this.checkedSet.add("checkAll");
        } else {
          results = Array.from(this.checkedSet);
        }
      } else {
        this.dataRecord.forEach((val, key) => {
          //如果存在子级
          if (val["children"].length > 0) {
            val["children"].forEach((children) => {
              let childrenKey =
                children[this.label] + "-" + children[this.nodekey];
              // 如果有被选中
              if (this.checkedSet.has(childrenKey)) {
                results.push(children);
              }
            });
          } else {
            //不存在子级，便把自身的推进去
            let itemKey = val[this.label] + "-" + val[this.nodekey];
            if (this.checkedSet.has(itemKey)) {
              results.push(val);
            }
          }
        });
        return results;
      }
    },
  },
};
</script>
<style lang='stylus' scoped>
.viewport {
  // min-width 200px
  position: relative;
  overflow-y: auto;

  .virtual-list {
    position: relative;
    left: 0;
    padding-right: 10px;
    overflow: hidden;

    .item {
      box-sizing: border-box;
      white-space: nowrap;
    }
  }

  .scrollToTop {
    position: fixed;
    bottom: 10px;
    right: 20px;
    border-radius: 50%;
    width: 30px;
    height: 30px;
    color: rgb(64, 158, 255);
    line-height: 30px;
    background: #fff;
    text-align: center;
    font-size: 20px;
    transform: rotate(90deg);
    cursor: pointer;
    box-shadow: rgba(0, 0, 0, 0.3) 1px 0px 2px 0px;
  }
}
</style>
<style scoped>
.check-box {
  display: inline-block;
  border: 1px solid #dcdfe6;
  width: 14px;
  height: 14px;
  background: #fff;
  border-radius: 2px;
  position: relative;
  line-height: 14px;
  text-align: center;
}
.el-icon-check {
  background-color: #409eff;
  border-color: #409eff;
  color: #fff;
}
.el-icon-caret-right {
  transition: all 0.5s ease;
}
</style>