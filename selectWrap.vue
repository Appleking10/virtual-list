<!--
  功能：功能描述
  作者：jinyini
  时间：2020年04月22日 15:10:56
  版本：v1.0
  应用名称：
-->
<template>
  <div class="container-popup">
    <div class="select-wrap">
      <slot name="selectSlot"></slot>
      <div class="select-all-wrap" v-click-outside="hidePanel">
        <!-- 点击input弹出下拉框 -->
        <div class="select-input-wrap" @click="roadNamePopup(!showPopup)">
          <input type="text" readonly="readonly" v-model="inputRecord" />
          <em :class="showPopup?'el-icon-caret-top':'el-icon-caret-bottom'"></em>
        </div>
        <!-- 下拉框 -->
        <div class="select-content-wrap" v-show="showPopup && transferData">
          <mtex-virtualList
            v-loading="loading"
            :isCheck="true"
            :itemH="26"
            :showNum="5"
            :datas="transferData"
            :remainNum="5"
            :filter="filterOpen"
            :label="label"
            :nodekey="nodekey"
            ref="transfer"
            v-if="showTranferFlag"
          >
            <span
              slot="singleList"
              slot-scope="{item}"
              style="display:inline-block;height:26px;"
            >{{item["parentKey"]?item["parentKey"].split('-')[0]:''}}: {{item[label]}}</span>
            <span slot="main" slot-scope="{main}">{{main.split("-")[0]}}</span>
            <span
              slot="sub"
              slot-scope="{sub}"
              style="display:inline-block;height:26px;"
            >{{sub[label]}}</span>
          </mtex-virtualList>
          <div v-else>暂无数据</div>

          <div class="select-bottom-wrap">
            <el-input
              placeholder="请输入内容"
              v-model="state"
              size="mini"
              @focus="addPYtag"
              @input="querySearch"
            >
              <i slot="prefix" class="el-input__icon el-icon-search"></i>
            </el-input>
            <button @click="confirm" class="select-btn">确定</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
const clickOutside = {
  // 初始化指令
  bind(el, binding, vnode) {
    function clickHandler(e) {
      // 这里判断点击的元素是否是本身，是本身，则返回
      if (el.contains(e.target)) {
        return false;
      }
      // 判断指令中是否绑定了函数
      if (binding.expression) {
        // 如果绑定了函数 则调用那个函数，此处binding.value就是handleClose方法
        binding.value(e);
      }
    }
    // 给当前元素绑定个私有变量，方便在unbind中可以解除事件监听
    el.__vueClickOutside__ = clickHandler;
    document.addEventListener("click", clickHandler);
  },
  update() {},
  unbind(el, binding) {
    // 解除事件监听
    document.removeEventListener("click", el.__vueClickOutside__);
    delete el.__vueClickOutside__;
  },
};
import virtualList from "../../../plugin2xCom/transfer/virtual-list/virtual-list.vue";
import pinyin from "../../../plugin2xCom/commonTools/pinyin.js";
export default {
  components: {
    "mtex-virtualList": virtualList,
  },
  data() {
    return {
      showPopup: false, //下拉框显示控制
      transferData: [], //传入虚拟列表的数据
      state: "", //搜索输入框的值
      Pinyin: pinyin, //引入的拼音方法
      inputRecord: "", //下拉框的显示值
      addPYFlag: false, //初始化加入拼音标识的字段
      resultRecord: null, //记录上一次选中
      nodekey: this.paramObj.nodeKey,
      label: this.paramObj.label,
      filterOpen: false,
      loading: false,
      showTranferFlag: false,
    };
  },
  directives: {
    clickOutside,
  },
  props: {
    paramObj: Object,
  },
  created() {
    this.transferDataRecord = null;
  },
  mounted() {
    let datamap = new Map();
    this.loading = false;
    if (this.paramObj && this.paramObj.data) {
      this.paramObj["data"].forEach((e) => {
        let key = `${e[this.label]}-${e[this.nodekey]}`;
        datamap.set(key, e);
      });

      this.transferData = datamap;
      this.transferDataRecord = datamap;
      this.$nextTick(() => {
        this.showTranferFlag = true;
      });
    } else {
      this.showTranferFlag = false;
    }
  },
  methods: {
    //模糊搜索
    querySearch() {
      let results = [];

      //如果是多级菜单的关键词检索

      //如果有搜索词
      if (this.state) {
        // 将搜索词转成拼音
        let py = this.Pinyin.convertPinyin(this.state.toLowerCase());
        if (!Array.isArray(this.transferDataRecord)) {
          //在py字段搜索这个拼音，符合返回
          this.transferDataRecord.forEach((val, key) => {
            if (val["children"].length > 0) {
              results = results.concat(
                val["children"].filter((e) => {
                  e["parentKey"] = key;
                  return e["py"].toLowerCase().indexOf(py) > -1;
                })
              );
            } else {
              //
              if (val["py"].toLowerCase().indexOf(py) > -1) {
                results.push(val);
              }
            }
          });
        } else {
          this.transferDataRecord.forEach((item) => {
            if (item["py"].toLowerCase().indexOf(py) > -1) {
              results.push(item);
            }
          });
        }
        this.filterOpen = true;
      } else {
        results = this.transferDataRecord;
        this.filterOpen = false;
      }
      this.transferData = results;
    },
    //给标签加拼音标识
    addPYtag() {
      if (!this.addPYFlag) {
        this.loading = true;

        if (Array.isArray(this.transferDataRecord)) {
          setTimeout(() => {
            this.transferDataRecord = this.transferDataRecord.map((e) => {
              e["py"] = this.setPinyinConvert(e[this.label]);
              return e;
            });
            this.addPYFlag = true;

            this.loading = false;
          }, 0);
        } else {
          setTimeout(() => {
            this.transferDataRecord.forEach((val, key) => {
              if (val["children"].length > 0) {
                val["children"] = val["children"].map((e) => {
                  e["py"] = this.Pinyin.convertPinyin(e[this.label]);
                  return e;
                });
                this.transferDataRecord.set(key, val);
              } else {
                val["py"] = this.Pinyin.convertPinyin(val[this.label]);
                this.transferDataRecord.set(key, val);
              }
            });

            this.addPYFlag = true;
            this.loading = false;
          }, 0);
        }
      } else {
        this.loading = false;

        this.querySearch();
      }
    },
    //确认选中
    confirm() {
      if (this.transferData.size > 0 || this.transferData.length > 0) {
        let result = this.$refs.transfer.getSelectItem(); //获取选中结果

        let resultStr = []; //结果字符串
        let lsArr = []; //区域数组
        let lsStr = ""; //区域字符串
        let id = this.paramObj.id;
        result.forEach((item) => {
          resultStr.push(item[this.nodekey]);
          if (item["ls"]) {
            let ls = item["ls"];
            ls.name = item.name;
            lsArr.push(ls);
          }
        });
        resultStr = resultStr.join(",");
        lsStr = this.getLsString(lsArr);
        this.$emit("input", resultStr);
        this.$emit("change", resultStr, this.resultRecord, lsArr, lsStr, id);
        this.resultRecord = resultStr;
        this.inputRecord = `已选${result.length}项`;
      }
      this.showPopup = false;
    },
    getLsString(lsArr) {
      let ls = "";
      for (let i = 0; i < lsArr.length; i++) {
        let item = lsArr[i];
        for (let j = 0; j < item.length; j++) {
          ls += item[j].join(",") + ";";
        }
        ls.lastIndexOf(";") == ls.length - 1 &&
          (ls = ls.substring(0, ls.length - 1));
        ls += "@";
      }
      ls.lastIndexOf("@") == ls.length - 1 &&
        (ls = ls.substring(0, ls.length - 1));
      return ls;
    },
    //道路名称弹窗
    roadNamePopup(visible) {
      if (this.showPopup) {
        this.confirm();
      }
      this.showPopup = visible;
    },
    hidePanel() {
      if (this.showPopup) {
        this.confirm();
        this.showPopup = false;
      }
    },
  },
};
</script>
<style scoped>
.select-all-wrap {
  position: relative;
}
.select-wrap {
  display: flex;
  flex-direction: row;
  justify-self: flex-start;
  align-content: center;
}
.select-input-wrap {
  width: 100px;
  height: 26px;
  background: rgba(255, 255, 255, 1);
  border-radius: 2px;
  border-width: 1px;
  border-style: solid;
  padding-left: 5px;
  line-height: 26px;
  display: flex;
  align-items: center;
  border-color: rgb(204, 204, 204);
  position: relative;
}
.select-input-wrap > input {
  width: 100%;
  height: 100%;
  border: none;
  outline: none;
}
.select-input-wrap > em {
  position: absolute;
  right: 5px;
  top: 7px;
}

.select-content-wrap {
  min-width: 170px;
  z-index: 999;
  background-color: #fff;
  position: absolute;
  top: 30px;
  left: 0px;
  padding: 10px;
  overflow: hidden;
  background: rgba(255, 255, 255, 1);
  box-shadow: 2px 2px 6px 2px rgba(8, 47, 77, 0.3);
  border-radius: 2px;
}
.select-content-wrap::after {
  content: "";
  display: block;
  clear: both;
}
.select-bottom-wrap {
  max-height: 180px;
  margin: 5px auto;
  width: 100%;
  box-sizing: border-box;
  overflow-y: hidden;
  text-align: left;
  display: flex;
  align-items: center;
}
.select-bottom-wrap >>> .el-input {
  padding-right: 5px;
}
.select-bottom-wrap >>> .el-input__inner {
  height: 24px;
  line-height: 24px;
}
.select-btn {
  width: 48px;
  height: 24px;
  font-size: 12px;
  line-height: 22px;
  color: white;
  background: rgba(56, 164, 255, 1);
  border-radius: 2px;
  border: 1px solid rgba(56, 164, 255, 1);
}
</style>