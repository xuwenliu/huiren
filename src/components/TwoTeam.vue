<template>
  <div class="team">
    <div class="team-top">
      <header>
        <span class="title">UMB ARENA</span>
        <span class="el-icon-setting"></span>
      </header>
      <div
        :style="{ height: isPC ? 'calc(50vh - 64px)' : 'calc(100vh - 64px)' }"
        class="dialog-box"
      >
        <div class="dialog-box-left" v-if="isPC">
          <div class="zsk">
            <div class="title">知识库问答</div>
            <div id="zskBox" class="content">
              <div v-for="(item, index) in dialog1Copy" :key="index">
                <div class="zsk-item">
                  <div class="fl-r desc msg msg-right">{{ item }}</div>
                </div>
              </div>
            </div>
            <!-- <div class="new-start">
                    <span class="new-start-item">重新开始</span>
            </div>-->
            <div class="search-line">
              <!-- <el-input placeholder="请输入您想问的" v-model.trim="dialog1"></el-input>&nbsp;&nbsp;
              <el-button type="primary" @click="seek1">询问</el-button>-->
              <el-input
                suffix-icon
                placeholder="请输入您想问的..."
                v-model.trim="dialog1"
              ></el-input>
              <span @click="seek1" class="el-icon-s-promotion"></span>
            </div>
          </div>
          <div class="charts">
            <EchartsItem
              v-if="chartTableData.part.length"
              ref="echarts"
              :option="opts1"
            />
          </div>
          <div style="padding: 0 48px 8px 48px">
            <table v-if="chartTableData.part.length" border="1" width="100%">
              <tr>
                <th>对话回合</th>
                <td v-for="(item, index) in chartTableData.part" :key="index">
                  {{ item }}
                </td>
              </tr>
              <!-- <tr>
                <th>当前对话</th>
                <td></td>
                <td></td>
                <td></td>
                <td></td>
                <td></td>
                <td></td>
                <td>5</td>
              </tr>-->
              <tr>
                <th>标准话术/情绪中值</th>
                <td v-for="(item, index) in chartTableData.verger" :key="index">
                  {{ item }}
                </td>
              </tr>
              <tr>
                <th>情绪波动</th>
                <td v-for="(item, index) in chartTableData.mood" :key="index">
                  {{ item }}
                </td>
              </tr>
            </table>
          </div>
        </div>

        <div class="dialog-box-right">
          <div class="header">
            <!-- <el-select size="mini" v-model="gptValue" placeholder="请选择">
                                <el-option
                                  v-for="item in options"
                                  :key="item.value"
                                  :label="item.label"
                                  :value="item.value"
                                ></el-option>
            </el-select>-->
            <span class="title">ChatGPT 4o</span>

            <span class="desc">客服/用户切换</span>
            <el-switch
              active-value="user"
              inactive-value="person"
              v-model="switchValue"
              active-color="#eee"
              inactive-color="#eee"
              disabled
            ></el-switch>
            <!-- <el-switch
                    active-value="person"
                    inactive-value="user"
                    v-model="switchValue2"
                    active-color="#eee"
                    inactive-color="#eee"
                    disabled
                  ></el-switch>
            <span class="desc">阶段数据</span>-->
            <!-- <el-link type="danger" @click="showEnd = true">结束本轮对话</el-link> -->
          </div>
          <div id="dialogBox" class="content">
            <div class="content-l">
              <div
                v-for="(item, index) in messageList"
                :key="index"
                class="dialog-item"
              >
                <div class="left">
                  <div class="user">
                    <div>AI用户</div>
                    <span>{{ item.yonghu }}</span>
                  </div>
                </div>
                <div
                  class="right"
                  :style="{ visibility: item.user ? 'visible' : 'hidden' }"
                >
                  <div class="right-item">
                    <div class="person">
                      <div>人类客服</div>
                    </div>
                    <div class="ai">
                      <div>AI客服</div>
                      <span>{{ item.user }}</span>
                    </div>
                  </div>
                </div>
                <div v-if="isPC && item.chartDetail" class="asideBox">
                  <div>
                    <span>客服会话：</span>
                    <span>【 R {{ index + 1 }} 】</span>
                  </div>
                  <div>
                    <span>ID：</span>
                    <span>{{ item.chartDetail.subInputId }}</span>
                  </div>
                  <div>
                    <span>知识库引用来源：</span>
                    <span>{{ item.chartDetail.kl?.[0] }}</span>
                  </div>
                  <div>
                    <span>话术对齐得分：</span>
                    <span>【85%】</span>
                  </div>
                  <div>
                    <span>话术对齐总分：</span>
                    <span>【70%】</span>
                  </div>
                  <br />
                  <div>
                    <span>用户意图识别 ID：</span>
                    <span>{{ item.chartDetail.subInputId }}</span>
                  </div>
                  <div>
                    <span>用户情绪识别：</span>
                    <span>{{ item.chartDetail.decisionMood }}</span>
                  </div>
                </div>
                <!-- <div class="result">
                      <p>评分表 {{ item }}</p>
                </div>-->
              </div>
              <div v-if="showEnd" class="dialog-end">
                <div class="main">
                  本轮对话已结束，以下是对本次对话的整体评价
                </div>
                <div class="desc">此处为本次圣诞的整体评价详情信息...</div>
                <!-- <br />
                    <el-button type="primary" size="mini" @click="showEnd = false"
                      >开始新对话</el-button
                >-->
              </div>
            </div>
            <div v-if="isPC" class="content-r"></div>
          </div>
          <div class="new-start">
            <span class="new-start-item" @click="restartDialog">New Chat</span>
            <span class="new-start-item" @click="stopDialog">挂 机</span>
            <!-- <span class="new-start-item" @click="showEnd = false"
                    >重新开始</span
            >-->
            <!-- <span class="new-start-item item-last">会话历史记录</span> -->
          </div>
          <div class="search-line">
            <el-input
              :disabled="showEnd"
              :placeholder="
                switchValue == 'user'
                  ? '你是用户，请在这里咨询AI客服相关问题'
                  : '你是一位客服人员，请在这里回复AI用户的问题'
              "
              v-model.trim="dialog2"
              suffix-icon
            ></el-input
            >&nbsp;&nbsp;
            <span @click="seek2" class="el-icon-s-promotion"></span>
            <!-- <el-button :disabled="showEnd" type="primary" @click="seek2">询问</el-button> -->
          </div>
        </div>
      </div>
    </div>
    <!-- <div class="team-bottom">
            <img style="width: 100vw; height: calc(50vh - 24px);" src="../assets/表格.png" alt="">
    </div>-->
    <div class="team-bottom" v-if="isPC">
      <TableCpn
        @delInfo="delInfo"
        :userInfo="userInfo"
        :oldUserInfo1="oldUserInfo1"
        :oldUserInfo2="oldUserInfo2"
      />
      <!-- <el-row>
        <el-col :span="16">
          <TableCpn />
        </el-col>
        <el-col :span="8">
          <div class="echarts-box">
                  <EchartsItem ref="echarts" :option="opts1" />
          </div>
          <div class="echarts-box">
                          <EchartsItem ref="echarts" :option="opts2" />
          </div>
        </el-col>
      </el-row> -->
    </div>
  </div>
</template>
      
<script>
import EchartsItem from "./Echarts.vue";
import TableCpn from "./TableCpn.vue";
import { tableData, getLineOption } from "./const";
export default {
  name: "two-team",
  components: {
    EchartsItem,
    TableCpn,
  },
  mounted() {
    // this.getUserInfo();
    // this.getOldUserInfo();
    // this.getChartData();
    this.$nextTick(() => {
      window.addEventListener("resize", this.handleResize);
      this.clientWidth = window.innerWidth;
      this.getRandomId();
    });
  },
  data() {
    return {
      clientWidth: 1200,
      dialog1: "",
      dialog1Copy: [],
      dialog2: "",
      options: [
        {
          value: "ChatGPT4.0",
          label: "ChatGPT4.0",
        },
        {
          value: "ChatGPT3.5",
          label: "ChatGPT3.5",
        },
        {
          value: "ChatGPT3.0",
          label: "ChatGPT3.0",
        },
      ],
      gptValue: "ChatGPT4.0",
      switchValue: "user",
      switchValue2: "user",
      messageList: [],
      showEnd: false,
      opts1: {},
      opts2: {},
      tableData,
      conversation_id: "",
      cidList: [],
      userInfo: {},
      oldUserInfo1: {},
      oldUserInfo2: {},
      randomId: "",
      chartTableData: {
        part: [],
        verger: [],
        mood: [],
      },
    };
  },
  methods: {
    getRandomId() {
      const userList = [
        "a5695fd5-f84b-45f1-9c61-12d33ada2bc7",
        "d6213300-89e8-4e31-835a-b5a8615ff374",
        "81191104-5425-4114-8d8c-6f9efc48260d",
        "26157a51-02d7-4aa1-bf19-a81470fb0139",
        "3b6a3c04-a84a-422c-a5a2-04085aa9e63a",
      ];
      const idx = () => Math.floor(Math.random() * 5);
      this.randomId = userList[idx()];
    },
    getTwoUserInfo() {
      if (!this.cidList.length) {
        this.getUserInfo();
      } else if (this.cidList.length == 1) {
        this.getUserInfo();
        this.getOldUserInfo(this.cidList[0], "first");
      } else {
        this.getUserInfo();
        const cid1 = this.cidList[this.cidList.length - 1];
        const cid2 = this.cidList[this.cidList.length - 2];
        //对话记录1
        this.getOldUserInfo(cid1, "first");
        //对话记录2
        this.getOldUserInfo(cid2, "second");
      }
    },
    handleResize() {
      this.clientWidth = window.innerWidth;
    },
    seek1() {
      return;
      if (!this.dialog1) {
        this.$message.error("请输入对话内容");
        return;
      }
      this.dialog1Copy.push(this.dialog1);
      setTimeout(() => {
        this.toBottom("zskBox");
      }, 100);
    },
    seek2() {
      if (!this.dialog2) {
        this.$message.error("请输入对话内容");
        return;
      }
      if (this.showEnd) {
        this.$message.error("请开始新对话");
        return;
      }
      if (this.switchValue == "user") {
        this.messageList.push({
          yonghu: this.dialog2,
        });
        this.sendChatAjax();
      } else {
        this.messageList.push({
          person: this.dialog2,
        });
      }
      setTimeout(() => {
        this.toBottom("dialogBox");
      }, 100);
      return;
      this.messageList.push({
        person: "你好，我是人工客服",
        ai: "你好，我是AI客服",
        user: "你好，我是用户",
      });
    },
    toBottom(id) {
      var dialogBox = document.getElementById(id);
      dialogBox.scrollTo({
        top: dialogBox.scrollHeight,
        behavior: "smooth",
      });
    },
    sendChatAjax() {
      var that = this;
      const { conversation_id, dialog2 } = this;
      this.$axios({
        method: "POST",
        headers: {
          authorization: "Bearer app-qMOb4h7wmCSfcMRkXclDGPGA",
        },
        url: "http://47.100.51.224/v1/chat-messages",
        data: {
          inputs: {},
          query: dialog2,
          response_mode: "blocking",
          conversation_id,
          user: that.randomId,
          files: [
            {
              type: "image",
              transfer_method: "remote_url",
              url: "https://cloud.dify.ai/logo/logo-site.png",
            },
          ],
        },
      })
        .then(function (res) {
          if (res.status == 200) {
            that.conversation_id = res.data?.conversation_id || "";
            that.$set(
              that.messageList[that.messageList.length - 1],
              "user",
              res.data?.answer
            );
            that.addSubYitu(res.data.conversation_id);
            that.getTwoUserInfo();
          }
        })
        .catch(function (err) {
          console.log(err);
        });
    },
    getUserInfo() {
      let that = this;
      this.$axios
        .get(
          "http://47.100.51.224:8080/api/user-profile/user?userId=" +
            that.randomId
        )
        .then(function (res) {
          if (res.status == 200) {
            that.userInfo = res.data || {};
          }
        })
        .catch(function (err) {
          console.log(err);
        });
    },
    getOldUserInfo(cid, part) {
      let that = this;
      this.$axios
        .get(
          "http://47.100.51.224:8080/api/user-profile/oldUserProfile?userId=" +
            that.randomId +
            "&cId=" +
            cid
        )
        .then(function (res) {
          if (res.status == 200) {
            if (part == "first") {
              that.oldUserInfo1 = res.data || {};
            } else if (part == "second") {
              that.oldUserInfo2 = res.data || {};
            }
          }
        })
        .catch(function (err) {
          console.log(err);
        });
    },
    addSubYitu(cid) {
      let that = this;
      //   that.getChartData([
      //     { decisionMood: "中性" },
      //     { decisionMood: "积极" },
      //     { decisionMood: "犹豫" },
      //     { decisionMood: "失望不满" },
      //     { decisionMood: "中性" },
      //     { decisionMood: "积极" },
      //     { decisionMood: "犹豫" },
      //     { decisionMood: "失望不满" }
      //   ]);
      this.$axios
        .get(
          "http://47.100.51.224:8080/api/thoughts/dialogue/sortedSubYitu?cId=" +
            cid
        )
        .then(function (res) {
          if (res.status == 200) {
            that.getChartData(res.data || []);
          }
        })
        .catch(function (err) {
          console.log(err);
        });
    },
    getChartData(list = []) {
      const moodMap = {
        中性: 5,
        积极: 10,
        犹豫: 3,
        失望不满: 0,
      };
      const newList = list.reverse();
      this.$set(
        this.messageList[list.length - 1],
        "chartDetail",
        newList[list.length - 1] || {}
      );
      const xData = newList.map((ele, idx) => "R" + (idx + 1));
      const yData = [
        newList.map(() => 5),
        newList.map((ele) => moodMap[ele.decisionMood]),
      ];
      this.chartTableData = {
        part: xData,
        verger: yData[0],
        mood: yData[1],
      };
      this.opts1 = getLineOption({
        title: '用户情绪反馈表',
        type: "line",
        legend: ["标准话术/情绪中值", "情绪波动"],
        yData,
        xData,
      });
      // this.opts2 = getLineOption({
      //   title: "五虎上将",
      //   type: "bar",
      //   legend: ["关云长", "张翼德", "赵子龙"],
      //   yData,
      //   xData,
      // });
    },
    delInfo() {
      this.oldUserInfo1.userInfo = {};
      this.oldUserInfo2.userInfo = {};
    },
    stopDialog() {
      if (!this.messageList.length) {
        this.$message.error("暂未开启对话");
        return;
      }
      this.showEnd = true;
      // cid 1
      this.conversation_id && this.cidList.push(this.conversation_id);
      this.conversation_id = "";
    },
    restartDialog() {
      this.messageList = [];
      this.chartTableData = {
        part: [],
        verger: [],
        mood: [],
      };
      this.showEnd = false;
    },
  },
  computed: {
    isPC() {
      return this.clientWidth > 800;
    },
  },
  watch: {
    showEnd() {
      setTimeout(() => {
        this.toBottom("dialogBox");
      }, 100);
    },
  },
};
</script>
 
<style lang="less">
body {
  margin: 0; // overflow: hidden;
  .el-input__inner {
    background: #eee;
    border: 1px solid #eee;
    &:focus {
      border: 1px solid #eee;
    }
    &:hover {
      border: 1px solid #eee;
    }
  }
  // .el-input__inner {
  //   background: #eee;
  //   &:focus {
  //     border: 1px solid #eee;
  //   }
  // }
}
</style>
<style lang="less" scoped>
.team {
  background: #f2f5f6;
  padding: 0 12px;
  .team-top {
    height: calc(50vh - 8px);
  }
  .team-bottom {
    height: 50vh;
  }
  header {
    height: 48px;
    // border-bottom: 1px solid #0068b85d;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    .title {
      font-size: 24px;
      font-weight: 900;
      color: #0e69b6;
    }
    .el-icon-setting {
      font-size: 32px;
      font-weight: 900;
      color: #0e69b6;
      margin: 0 8px 0 16px;
    }
  }
  .dialog-box {
    height: calc(50vh - 64px);
    display: flex;
    column-gap: 8px;
    .desc {
      font-size: 14px;
      color: #606266;
    }
    .content {
      .msg {
        position: relative;
        width: fit-content;
        max-width: 250px;
        &::after {
          content: "";
          position: absolute;
          width: 10px;
          height: 10px;
          transform: rotate(45deg);
          top: 2px;
        }
      }
      .msg-left {
        padding: 10px 12px;
        background: #ffefb2;
        border-radius: 2px;
        margin: 0 60px 0 20px;
        &::after {
          background: #ffefb2;
          left: -5px;
        }
      }
      .msg-right {
        color: #fff;
        padding: 10px 12px;
        background: #0e69b6;
        border-radius: 2px;
        margin: 0 20px 0 60px;
        &::after {
          background: #0e69b6;
          right: -5px;
        }
      }
    }
    .dialog-box-left,
    .dialog-box-right {
      display: flex;
      flex-direction: column;
      background: #fff;
      .title {
        height: 48px;
      }
      /deep/.el-switch__core {
        background: #0e69b6 !important;
      }
      .content {
        flex: 1;
        padding: 20px 0;
        border-bottom: 1px solid #f0f1f4;
      }
      .search-line {
        display: flex;
        padding: 0 16px;
        position: relative;
        .el-icon-s-promotion {
          color: #0e69b6;
          font-size: 24px;
          cursor: pointer;
          transform: translateY(8px);
          position: absolute;
          left: calc(100% - 64px);
        }
      }
      .new-start {
        padding: 8px 0 8px 16px;
      }
      .new-start-item {
        color: #0067b8;
        border: 1px solid #eee;
        padding: 2px 4px;
        font-size: 13px;
        border-radius: 4px;
        margin-right: 8px;
        cursor: pointer;
        &:hover {
          background-color: #0e69b6;
          color: #fff;
        }
      }
      .item-last {
        float: right;
      }
    }
    .dialog-box-left {
      flex: 42;
      //   border-right: 1px solid #0068b85d;

      .title {
        color: #0067b8;
        font-weight: 900;
        height: 48px;
        padding-left: 16px;
        line-height: 48px;
        border-bottom: 1px solid #f0f1f4;
      }
      .content {
        height: 72px;
        border-bottom: none;
        // overflow-y: scroll;
        .zsk-item {
          overflow: hidden;
          margin-bottom: 12px;
        }
      }
      .charts {
        flex: 1;
        transform: translate(20px, 20px);
      }
      .el-icon-s-promotion {
        left: calc(100% - 52px) !important;
      }
    }
    .dialog-box-right {
      flex: 60;
      .header {
        height: 48px;
        padding-left: 16px;
        line-height: 48px;
        border-bottom: 1px solid #f0f1f4;
        .title {
          color: #0067b8;
          font-weight: 900;
        }
        .el-switch,
        .desc {
          float: right;
        }
        .el-switch {
          transform: translateY(14px);
        }
        .el-switch.is-disabled {
          opacity: 1;
        }
        .desc {
          margin: 0 16px 0 16px;
        }
      }
      .content {
        overflow-y: scroll;
        padding: 0 40px 40px 20px;
        font-size: 12px;
        .dialog-end {
          margin-top: 20px;
          border-radius: 10px;
          background: #edf5f8;
          padding: 20px;
          .main,
          .desc {
            margin-bottom: 8px;
          }
          .main {
            color: #206ccf;
            font-weight: 900;
            font-size: 14px;
          }
          .desc {
            font-size: 12px;
          }
        }
        .dialog-item {
          padding: 20px 0 20px 20px;
          border-bottom: 1px dotted #c9cdd4; // overflow: hidden;
          position: relative;
          .asideBox {
            position: absolute;
            top: 0;
            background: #eef3f9;
            width: 33%;
            height: 100%;
            right: 0;
            transform: translateX(100%);
            padding: 20px;
            margin-bottom: 8px;
          }
          .right {
            overflow: hidden;
            .right-item {
              max-width: 260px;
              margin-top: 16px;
              float: right; //   margin-right: 40px;
              //   .el-tag {
              //     margin: 8px 0;
              //   }
              border-radius: 10px;
              border: 1px solid #0a50a9;
              overflow: hidden;
              margin-right: 32px;
              .person {
                background: #0a50a9;
                padding: 16px 10px;
                div {
                  width: fit-content;
                  padding: 5px 10px;
                  background: #fff;
                  border-radius: 20px;
                  color: #0a50a9;
                  margin-bottom: 10px;
                }
              }
              .ai {
                padding: 16px 10px;
                div {
                  width: fit-content;
                  padding: 5px 10px;
                  background: #0a50a9;
                  border-radius: 20px;
                  color: #fff;
                  margin-bottom: 10px;
                }
              }
            }
          }
          .left {
            overflow: hidden;
            .user {
              float: left;
              width: 200px;
              border-radius: 10px;
              background: #f2f2f2;
              padding: 10px;
              div {
                width: fit-content;
                padding: 5px 10px;
                background: #fff;
                border-radius: 20px;
                color: #0a50a9;
                margin-bottom: 10px;
              }
            }
          }
        }
        display: flex;
        .content-l {
          flex: 3;
        }
        .content-r {
          flex: 1;
          height: calc(100%);
        }
      }
    }
  }
  .echarts-box {
    width: 100%;
    height: 320px;
  }
  .fl-r {
    float: right;
  }
  .table-wrapper {
    overflow-x: scroll;
  }
  table,
  tr,
  td,
  th {
    border-collapse: collapse;
    text-align: center;
    font-size: 12px;
    border: 1px solid rgb(184, 182, 182);
  }
  th {
    width: 140px;
    font-weight: 400;
  }
}
</style>
