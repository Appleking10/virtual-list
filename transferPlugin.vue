<!--
  功能：功能描述
  作者：jinyini
  时间：2020年04月22日 15:10:56
  版本：v1.0
  应用名称：
-->
<template>
  <div class="container-popup" v-if="showPopup" ref="transferWrap">
    <div class="transfer-wrap">
      <div class="transfer-left-wrap">
        <div class="showNum">
          <span style="color: #333; font-size: 14px">待选择</span>
          <span>{{ transfer_L_check }}/{{ transfer_L_Total }}</span>
        </div>
        <div class="select-bottom-wrap">
          <el-input
            placeholder="请输入内容"
            style="padding: 5px"
            v-model="search_L_val"
            size="mini"
            @focus="addPYtag"
            @input="
              querySearch(
                search_L_val,
                'transfer_L_Map',
                'transfer_L_DataRecord'
              )
            "
          >
            <i slot="prefix" class="el-input__icon el-icon-search"></i>
          </el-input>
        </div>
        <mtex-virtualList
          style="padding: 0 5px"
          :isCheck="true"
          :itemH="26"
          :itemW="250"
          :showNum="8"
          :datas="transfer_L_Map"
          :remainNum="8"
          :label="label"
          :nodekey="nodekey"
          ref="transferLeft"
          v-if="transfer_L_Map"
          @checked="leftChecked"
        >
          <span
            slot="singleList"
            slot-scope="{ item }"
            style="display: inline-block; height: 26px"
          >
            {{ item["parentKey"] ? item["parentKey"].split("-")[0] : "" }}:
            {{ item[label] }}
          </span>
          <span slot="main" slot-scope="{ main }">{{
            main.split("-")[0]
          }}</span>
          <span
            slot="sub"
            slot-scope="{ sub }"
            style="display: inline-block; height: 26px"
            >{{ sub[label] }}</span
          >
        </mtex-virtualList>
        <div v-else style="padding: 0 5px">暂无数据</div>
      </div>
      <div class="transfer-buttons-wrap">
        <div
          class="transfer-button"
          v-bind:class="{ 'btn-check': checked_L_Flag }"
          @click="removeRCheckedData"
        >
          <span class="el-icon-arrow-right"></span>
        </div>
        <div
          class="transfer-button"
          v-bind:class="{ 'btn-check': transferCheck_R_Set.size > 0 }"
          @click="removeLCheckedData"
        >
          <span class="el-icon-arrow-left"></span>
        </div>
      </div>
      <div class="transfer-right-wrap">
        <div class="showNum">
          <span style="color: #333; font-size: 14px">已选择</span>
          <span>{{ transfer_R_check }}/{{ transfer_R_Total }}</span>
        </div>
        <div class="select-bottom-wrap">
          <el-input
            style="padding: 5px"
            placeholder="请输入内容"
            v-model="search_R_val"
            size="mini"
            @input="
              querySearch(
                search_R_val,
                'transfer_R_Map',
                'transfer_R_DataRecord'
              )
            "
          >
            <i slot="prefix" class="el-input__icon el-icon-search"></i>
          </el-input>
        </div>
        <mtex-virtualList
          style="padding: 0 5px"
          :isCheck="true"
          :itemH="26"
          :itemW="250"
          :showNum="8"
          :datas="transfer_R_Map"
          :remainNum="8"
          :label="label"
          :nodekey="nodekey"
          ref="transferRight"
          @checked="rightChecked"
          v-if="transfer_R_Map.size > 0 || transfer_R_Map.length > 0"
        >
          <span
            slot="singleList"
            slot-scope="{ item }"
            style="display: inline-block; height: 26px"
          >
            {{ item["parentKey"] ? item["parentKey"].split("-")[0] : "" }}:
            {{ item[label] }}
          </span>
          <span slot="main" slot-scope="{ main }">{{
            main.split("-")[0]
          }}</span>
          <span
            slot="sub"
            slot-scope="{ sub }"
            style="display: inline-block; height: 26px"
            >{{ sub[label] }}</span
          >
        </mtex-virtualList>
        <div v-else style="padding: 0 5px">暂无数据</div>
      </div>
    </div>
    <div class="bottom-button-wrap">
      <el-button type="primary" size="mini" @click="getTransferData"
        >确认</el-button
      >
      <el-button type="info" size="mini" @click="transferPopup(false)"
        >取消</el-button
      >
    </div>
  </div>
</template>
<script>
import virtualList from "./virtual-list/virtual-list-update.vue";
import pinyin from "../commonTools/pinyin.js";
export default {
  components: {
    "mtex-virtualList": virtualList,
  },
  data() {
    return {
      showPopup: false, //下拉框显示控制
      transfer_L_Map: null, //传入左边虚拟列表的数据
      search_L_val: "", //左边搜索输入框的值
      search_R_val: "", //右边搜索输入框的值
      checked_L_Flag: false, // >按钮的高亮控制
      Pinyin: pinyin, //引入的拼音方法
      inputRecord: "", //1.下拉框的显示值
      addPYFlag: false, //初始化加入拼音标识的字段
      resultRecord: null, //2.记录上一次选中
      showTranferFlag: false, //数据准备好可以显示
      transfer_L_Total: 0,
      transfer_L_check: 0,
      transfer_R_check: 0,
      transfer_R_Total: 0,
      nodekey: this.paramObj.nodeKey,
      label: this.paramObj.label,
      filterOpen: false, //3.
      loading: false,
      roadArray: [],
      roadName: "",
      roadNameMap: null, //4.
      transfer_R_Map: [],
      transferCheck_L_Set: new Set(),
      transferCheck_R_Set: new Set(),
    };
  },
  props: {
    paramObj: Object,
  },
  created() {
    this.transfer_L_DataRecord = null;
    this.transfer_R_DataRecord = null;
  },
  mounted() {
    // let datamap = new Map();
    // this.loading = true;

    // this.transfer_L_Map = datamap;
    // this.transfer_L_DataRecord = new Map(datamap);
    // this.$nextTick(() => {
    //   this.showTranferFlag = true;
    // });
    this.transfer_L_Map = this.paramObj.data; //地图

    this.transfer_L_DataRecord = new Map(this.paramObj.data); //地图
    this.transferPopup(true);
    this.$nextTick(() => {
      let element = this.$refs.transferWrap;
      this.$popupLoading({
        show: true,
        element,
        message: "加载数据中",
      });
      this.addPYtag();
    });
  },
  methods: {
    //选中监听
    leftChecked(checkedSet) {
      // console.log(checkedSet);
      if (checkedSet.size) {
        this.checked_L_Flag = true;
      } else {
        this.checked_L_Flag = false;
      }
      this.transfer_L_check = this.$refs.transferLeft.getSelectItem().length;
      this.transferCheck_L_Set = new Set(checkedSet);
    },
    //点击》按钮
    removeRCheckedData() {
      if (this.transferCheck_L_Set.size) {
        let checkedData = this.$refs.transferLeft.getSelectItem();
        this.transfer_R_Total = checkedData.length;
        let checkedMap = new Map();
        let key;
        if (this.transferCheck_L_Set.has("checkAll")) {
          this.transfer_R_Map = new Map(this.transfer_L_Map);
          this.transfer_R_DataRecord = new Map(this.transfer_L_Map);
        } else {
          checkedData.forEach((item, index) => {
            key = item["parentKey"];
            if (checkedMap.has(key) == false) {
              checkedMap.set(key, {
                children: [],
              });
            }

            checkedMap.get(key)["children"].push(item);
          });
          this.transfer_R_Map = checkedMap;
          this.transfer_R_DataRecord = checkedMap;
        }
      }
    },
    // 点击《按钮
    removeLCheckedData() {
      if (this.transferCheck_R_Set.size) {
        let checkedData = this.$refs.transferRight.getSelectItem();
        let key, children;

        if (this.transferCheck_R_Set.has("checkAll")) {
          //先判断是否有全选
          this.transfer_R_Map = new Map();
          this.transferCheck_R_Set = new Set();
        } else {
          this.transfer_R_Map.forEach((val, parentkey) => {
            //判断一级菜单有没有全选
            if (this.transferCheck_R_Set.has(parentkey)) {
              //清空二级，且删掉本身一级菜单
              val["children"].forEach((item) => {
                this.transferCheck_R_Set.delete(
                  `${item[this.label]}-${item[this.nodekey]}`
                );
              });
              this.transferCheck_R_Set.delete(parentkey);
              this.transfer_R_Map.delete(parentkey);
            }
          });
          //再去处理单条的
          checkedData.forEach((item) => {
            key = `${item[this.label]}-${item[this.nodekey]}`;
            if (this.transferCheck_R_Set.has(key)) {
              children = this.transfer_R_Map.get(item["parentKey"])["children"];
              let index = children.findIndex((e) => {
                return key == `${e[this.label]}-${e[this.nodekey]}`;
              });
              children.splice(index, 1);
              this.transferCheck_R_Set.delete(key);
            }
          });
        }

        this.$refs.transferRight.checkedSet = new Set(this.transferCheck_R_Set);
        this.transfer_R_Map = new Map(this.transfer_R_Map);
      }
    },
    rightChecked(checkedSet) {
      this.transferCheck_R_Set = new Set(checkedSet);
      this.transfer_R_check = this.$refs.transferRight.getSelectItem().length;
    },
    //模糊搜索
    /*
    @param：搜索值，
    */
    querySearch(searchVal, resultMapName, dataName) {
      let results = [];

      //如果有搜索词
      if (searchVal) {
        // 将搜索词转成拼音
        let py = this.Pinyin.convertPinyin(searchVal.toLowerCase());
        if (!Array.isArray(this[dataName])) {
          //在py字段搜索这个拼音，符合返回
          this[dataName].forEach((val, key) => {
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
          this[dataName].forEach((item) => {
            if (item["py"].toLowerCase().indexOf(py) > -1) {
              results.push(item);
            }
          });
        }
        this.filterOpen = true;
      } else {
        results = this[dataName];
        this.filterOpen = false;
      }
      this[resultMapName] = results;
    },
    //给标签加拼音标识
    addPYtag() {
      if (!this.addPYFlag) {
        // this.loading = true;

        if (Array.isArray(this.transfer_L_DataRecord)) {
          setTimeout(() => {
            this.transfer_L_DataRecord = this.transfer_L_DataRecord.map((e) => {
              e["py"] = this.setPinyinConvert(e[this.label]);
              return e;
            });
            this.addPYFlag = true;

            this.loading = false;
          }, 0);
        } else {
          setTimeout(() => {
            this.transfer_L_DataRecord.forEach((val, key) => {
              if (val["children"].length > 0) {
                val["children"] = val["children"].map((e) => {
                  e["py"] = this.Pinyin.convertPinyin(e[this.label]);
                  return e;
                });
                this.transfer_L_DataRecord.set(key, val);
              } else {
                val["py"] = this.Pinyin.convertPinyin(val[this.label]);
                this.transfer_L_DataRecord.set(key, val);
              }
            });
            // setTimeout(() => {}, 5000);
            this.addPYFlag = true;
            this.loading = false;

            this.$popupLoading({
              show: false,
              element: this.$refs.transferWrap,
            });
          }, 0);
        }
      } else {
        this.loading = false;

        this.querySearch(
          this.search_L_val,
          "transfer_L_Map",
          "transfer_L_DataRecord"
        );
      }
    },

    //获取选中
    getTransferData() {
      console.log(this.transfer_R_Map);
      let result = [];
      let resultStr = "";
      this.transfer_R_Map.forEach((val, key) => {
        result = result.concat(
          val["children"].map((e) => {
            return e[this.nodekey];
          })
        );
      });
      resultStr = result.join(",");
      console.log(resultStr);
      return resultStr;
    },
    //道路名称弹窗
    transferPopup(visible) {
      this.showPopup = visible;
    },
  },
};
</script>
<style scoped>
.container-popup {
  /* position: absolute; */
  /* top: 100px; */
  /* left: 100px; */
  padding: 20px;
  border: 1px solid #ebebeb;
  border-radius: 3px;
  width: 400px;
  background: #fff;
  /* border: 1px solid darkgray; */
}

.transfer-wrap {
  display: flex;
  align-items: center;
}

.transfer-buttons-wrap {
  margin: 0 10px;
}

.transfer-buttons-wrap > .transfer-button {
  width: 36px;
  height: 36px;
  line-height: 36px;
  text-align: center;
  margin: 10px auto;
  border-radius: 100%;
  color: #c0c4cc;
  background-color: #f5f7fa;
  border: 1px solid #c0c4cc;
  cursor: not-allowed;
}

.showNum {
  /* text-align: right; */
  color: rgb(143, 143, 147);
  height: 30px;
  line-height: 30px;
  padding: 0 5px;
  background: #f5f7fa;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.transfer-buttons-wrap > .btn-check {
  background-color: #409eff;
  color: #fff;
  border-color: #409eff;
  cursor: pointer;
}

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

.transfer-left-wrap,
.transfer-right-wrap {
  align-self: stretch;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  background: #fff;
  box-sizing: border-box;
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
  margin-bottom: 5px;
  width: 100%;
  box-sizing: border-box;
  overflow-y: hidden;
  text-align: left;
  display: flex;
  /* align-items: center; */
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

.bottom-button-wrap {
  display: flex;
  width: 100%;
  justify-content: flex-end;
  padding-right: 10px;
  margin-top: 10px;
  /* margin: 10px auto; */
  align-items: center;
}
</style>