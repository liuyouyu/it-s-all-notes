<template>
  <div class="detail-container">
    <Row type="flex" justify="end">
      <span>页面编辑中…</span>
    </Row>
    <div style="border-bottom: 1px dotted #EAEAEA;padding: 10px 0 10px 18px;">
      <Row type="flex" justify="space-between" class="detail-row" style="margin-top:10px;">
        <Col span="12">
          <span class="label-title">标题:</span>
          <Input v-model="editData.msTitle" style="width: 260px"/>
        </Col>
        <Col span="12">
          <span class="label-title" style="width:60px;">链接:</span>
          <Input v-model="editData.msUrl" style="width: 260px"/>
        </Col>
      </Row>
      <Row type="flex" justify="space-between" class="detail-row">
        <Col span="12">
          <span class="label-title">媒体:</span>
          <Select v-model="editData.msMedia" placeholder="全部" style="width: 260px">
            <Option
              v-for="item in mediaOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></Option>
          </Select>
        </Col>
        <Col span="12">
          <span class="label-title" style="width:60px;">发布日期:</span>
          <DatePicker
            :value="editData.msPublishTime"
            type="date"
            placement="bottom-end"
            @on-change="editData.msPublishTime=$event"
            split-panels
            style="width: 260px"
          ></DatePicker>
        </Col>
      </Row>
      <Row type="flex" justify="space-between" class="detail-row">
        <Col span="12">
          <span class="label-title">阅读量:</span>
          <Input v-model="editData.msReadVolume" style="width: 260px"/>
        </Col>
      </Row>
    </div>
    <Row type="flex" justify="space-between" class="detail-row">
      <span style="font-weight: bold;margin-right:10px;">扩展考核因子:</span>
      <Button
        type="primary"
        @click="computedAllResult"
        :disabled="!permission.PID_Fast_assessmentSystem_Assessment"
      >计算</Button>
    </Row>
    <Row type="flex" justify="start" class="detail-row" style="height:auto">
      <Col span="8" v-for="item in allFactorList">
        <span class="label-title" style="width: 80px;">{{item.configName}}</span>
        <Input type="number" v-model="item.configValue" style="width: 120px;margin-bottom:20px;"   v-if="item.configValueType==='1'"/>
        <Input   v-model="item.configValue" style="width: 120px;margin-bottom:20px;"   v-if="item.configValueType==='2'"/>
        <Select  v-model="item.configValue" placeholder="全部" style="width: 120px"  v-if="item.configValueType==='3'">
            <Option
              v-for="item2 in item.asFactorEnum"
              :key="item2.value"
              :label="item2.enumName"
              :value="item2.enumValue"
            ></Option>
          </Select>
        <Select  v-model="item.configValue" placeholder="全部" style="width: 120px"  v-if="item.configValueType==='4'">
            <Option
              v-for="item2 in item.asFactorEnum"
              :key="item2.value"
              :label="item2.enumName"
              :value="item2.enumValue"
            ></Option>
          </Select>
      </Col>
    </Row>
    <Table :columns="columnsPerson" :data="editData.personnel">
      <template slot-scope="{ row, index }" slot="scoreOperation">
        <Input
          v-model="row.ok"
          style="min-Width:10px;"
          v-on:on-blur="selectChange($event, index)"
          :disabled="!permission.PID_Fast_assessmentSystem_result"
        />
      </template>
      <template slot-scope="{ row, index }" slot="scoreOperation2">
        <i
          class="iconfont icon-list_delete_normal"
          :class="{righticon: index !== editData.personnel.length-1}"
          @click="deleteRow(index)"
        ></i>
        <i
          class="iconfont icon-list_add_normal"
          style="color:#0270D7;"
          v-if="index === editData.personnel.length-1"
          @click="addRow(index)"
        ></i>
      </template>
    </Table>
    <Row type="flex" justify="center" class="detail-row" style="margin-top:35px;">
      <Button type="primary" class="confirm-btn" @click="saveNew(editData, isUpdate)">保存</Button>
      <Button class="cancel-btn" @click="closeModal">取消</Button>
    </Row>
  </div>
</template>
<script>
export default {
  data() {
    return {
      isUpdate: false, //默认是新建。
      personLoading: false,
      // personOptions: [], //匹配的option
      // personOptionsAll: ["ABCDEFGHIG", "ABC", "ABCDEFGHIGKLMNNN", "ABCD"], //人员下拉框所有的option
      compoutedSorce: {
        msReadVolume: "1",
        selectFactorListModel: "100"
      },
      columnsPerson: [
        {
          title: "任务",
          key: "userTask",
          align: "center",
          minWidth: 10,
          // slot: 'taskSlot'
          render: (h, params) => {
            return h(
              "Select",
              {
                style: {
                  "min-width": "10px"
                },
                props: {
                  value: params.row.userTask ? params.row.userTask : "", //给下拉赋值
                  transfer: true
                },
                on: {
                  "on-change": value => {
                    // 无法数据双向绑定；手动绑定
                    console.log(params);
                    this.editData.personnel[params.index].userTask = value;
                  }
                }
              },
              this.selectTaskList.map(item => {
                return h("Option", {
                  props: {
                    value: item.label,
                    label: item.label
                  }
                });
              })
            );
          }
        },
        {
          title: "人员",
          key: "userName",
          minWidth: 10,
          align: "center",
          render: (h, params) => {
            return h(
              "Select",
              {
                style: {
                  "min-width": "10px"
                },
                props: {
                  value: params.row.userName,
                  transfer: true,
                  filterable: true,
                  remote: true,
                  loading: this.personLoading,
                  "remote-method": this.remoteMethodPerson
                },
                on: {
                  "on-change": value => {
                    // 无法数据双向绑定；手动绑定
                    this.editData.personnel[params.index].userName = value;
                  }
                }
              },
              this.personOptions.map(item => {
                return h("Option", {
                  props: {
                    value: item.value,
                    label: item.label
                  }
                });
              })
            );
          }
        },
        // {
        //   title: "得分",
        //   key: "assessScore",
        //   minWidth: 10,
        //   align: "center",
        //   slot: "scoreOperation"
        // },
        {
          title: "操作",
          minWidth: 10,
          key: "assessScore",
          align: "center",
          slot: "scoreOperation2"
        }
      ],
      selectTaskList: [
        {
          value: "0",
          label: "全部"
        }
      ],
      mediaOptions: [
        {
          value: "0",
          label: "全部"
        }
      ],
      selectFactorList: [
        {
          value: "0",
          label: "全部"
        }
      ], //得分下拉
      columnOptions: [
        {
          value: "0",
          label: "全部"
        },
        {
          value: "11",
          label: "111"
        },
        {
          value: "22",
          label: "22266"
        },
        {
          value: "33",
          label: "33366"
        },
        {
          value: "44",
          label: "44466"
        }
      ],
      taskOptions: [
        {
          value: "编辑",
          label: "编辑label"
        },
        {
          value: "采访",
          label: "采访label"
        }
      ],
      columnOptions: [
        {
          value: "0",
          label: "全部"
        },
        {
          value: "11",
          label: "111"
        },
        {
          value: "22",
          label: "22266"
        },
        {
          value: "33",
          label: "33366"
        },
        {
          value: "44",
          label: "44466"
        }
      ],
      // 所有考核因子
      allFactorList:[
        {
          configName:"11",
          configValue:"222",
          configValueType:"1"
        }
      ]
    };
  },
  props: {
    detailData: {
      type: Object
    },
    permission: {
      type: Object
    },
    personOptions: {
      type: Array
    }, //匹配的option
    personOptionsAll: {
      type: Array
    }
  },

  computed: {
    editData() {
      var objData = { ...this.detailData };

      console.log(objData);

      return objData;
    }
  },
  methods: {
    initResultTable() {
      // let score =
      //   parseInt(this.compoutedSorce.msReadVolume) *
      //   parseInt(this.editData.msReadVolume) *
      //   parseInt(this.compoutedSorce.selectFactorListModel);
      // this.editData.personnel.forEach((v, k, arr) => {
      //   v.assessScore = score;
      // });
      // this.$emit("scoreComputed", score);
      var that = this;
      JSON.parse(window.sessionStorage.getItem("allSelectResultList")).forEach(
        (v, k, arr) => {
          var obj = {
            title: v.asResultName,
            key: v.asResultId,
            align: "center",
            minWidth: 10,
            slot: "scoreOperation"
          };
          // thead
          that.columnsPerson.splice(this.columnsPerson.length - 1, 0, obj);

          // this.editData.personnel.forEach((v, k, arr) => {
          //   v.assessScore = score;
          // });

          // 考核结果
          that.editData.personnel.forEach((v2, k2, arr2) => {
            arr2[k2][v.asResultId] = v.asResultName;
            arr2[k2]["ok"] = "morenzhiaa";
          });
        }
      );

      console.log(that.editData.personnel);
    },
    computedAllResult() {},
    remoteMethodPerson(query) {
      if (query !== "") {
        this.personLoading = true;
        setTimeout(() => {
          this.personLoading = false;
          const list = this.personOptionsAll.map(item => {
            return {
              value: item.replace(/\([^\)]*\)/g, ""),
              label: item
            };
          });
          // console.log("list:",list)
          this.personOptions = list.filter(
            item => item.value.toLowerCase().indexOf(query.toLowerCase()) > -1
          );
        }, 200);
      } else {
        this.personOptions = [];
      }
    },
    selectChange(e, index) {
      // 无法数据双向绑定；手动绑定
      console.log("value:", e.target.value, index);
      switch (e.target.name) {
        case "addressInput":
          this.editData.msReprintMedia[index].mediaUrl = e.target.value;
          break;
        default:
          this.editData.personnel[index].assessScore = parseFloat(
            e.target.value
          );
          break;
      }

      console.log("editData:", this.editData);
    },

    saveNew(data, isupdate) {
      if (this.editData.manuscriptId !== "") {
        //   修改
        isupdate = true;
      } else {
        //   新增
        isupdate = false;
      }
      this.$emit("saveNew", data, isupdate);
    },
    closeModal() {
      this.$emit("closeModal");
    },
    deleteRow(index, type) {
      if (this.editData.personnel.length === 1) {
        this.$Message.success({
          content: "最后一项无法删除",
          duration: 2
        });
        return;
      }
      this.editData.personnel.splice(index, 1);
    },
    // 加号
    addRow(index) {
      let obj = {
        userTaskName: {},
        userName: "",
        assessScore: ""
      };
      console.log(this.editData.personnel);
      this.editData.personnel.push(obj);
      console.log(this.editData.personnel);

      // this.$forceUpdate();
    },
    initOptionsData() {
      let selectFactorList = JSON.parse(
        window.sessionStorage.getItem("selectFactorList")
      );
      this.selectFactorList = selectFactorList;
      // 任务下拉
      let selectTaskList = JSON.parse(
        window.sessionStorage.getItem("selectTaskList")
      );
      this.selectTaskList = selectTaskList;
      // 媒体
      // 媒体
      let mediaOptions = JSON.parse(
        window.sessionStorage.getItem("selectMediaList")
      );
      let mediaOptionsForFilter = [];
      mediaOptions.map((v, k, arr) => {
        if (v.channel === "WECHAT_ID") {
          mediaOptionsForFilter.push(v);
        }
      });
      this.mediaOptions = mediaOptionsForFilter;
    },
    initassessFactor() {
      let allFactorList = JSON.parse(
        window.sessionStorage.getItem("allAssessFactorList")
      );

      this.allFactorList=allFactorList;

      console.log(allFactorList)
    }
  },

  mounted() {
    this.initOptionsData();
    this.initassessFactor();
    this.initResultTable();
  }
};
</script>

<style lang="scss" scoped>
.detail-container {
  height: 546px;
  padding: 16px 16px 0;
  font-size: 14px;
  overflow-y: auto;
  .detail-row {
    color: #333333;
    height: 30px;
    margin: 20px 0;
    line-height: 30px;
    .label-title {
      font-weight: bold;
      width: 48px;
      text-align: right;
      margin-right: 22px;
      display: inline-block;
    }
    .more-wrap {
      width: calc(100% - 75px);
      display: inline-block;
    }
    .ivu-col {
      display: flex;
    }
  }
  .confirm-btn,
  .cancel-btn {
    width: 100px;
    height: 36px;
    font-size: 14px;
  }
  .confirm-btn {
    background: #0270d7;
    color: #ffffff;
    margin-right: 20px;
  }
  .text-btn {
    color: #3998ff;
    padding: 0;
  }
  .ivu-table {
    th,
    td {
      padding-left: 9px;
      background: #fff;
    }
  }
  .icon-list_delete_normal {
    margin: 0 15px;
    color: #d76554;
  }
  .righticon {
    margin-right: 40px;
  }
}
</style>