<template>
  <div class="contest-body">
    <el-row>
      <el-col :lg="24" :md="24" :xs="24">
        <el-card shadow>
          <div class="contest-title">
            <div slot="header">
              <span class="panel-title">{{ Contest.title }}</span>
            </div>
          </div>
          <div class="contest-time">
            <el-row>
              <el-col :md="12" :xs="24" class="left">
                <p>
                  <i aria-hidden="true" class="fa fa-hourglass-start"></i>
                  {{ $t('m.StartAt') }}：{{ Contest.start_time | localtime }}
                </p>
              </el-col>
              <el-col :md="12" :xs="24" class="right">
                <p>
                  <i aria-hidden="true" class="fa fa-hourglass-end"></i>
                  {{ $t('m.EndAt') }}：{{ Contest.end_time | localtime }}
                </p>
              </el-col>
            </el-row>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <div class="sub-menu">
      <el-tabs v-model="route_name" @tab-click="tabClick">
        <!--题目-->
        <el-tab-pane :disabled="contestMenuDisabled" lazy name="ContestProblemList">
          <span slot="label"><i aria-hidden="true" class="fa fa-list"></i>&nbsp;{{ $t('m.Problem') }}</span>
          <vxe-table
          ref="problemList"
          :data="problemList"
          auto-resize
          border="inner"
          stripe
          @cell-mouseenter="cellHover"
      >
        <vxe-table-column v-if="isAuthenticated" title="" width="30">
          <template v-slot="{ row }">
            <template v-if="isGetStatusOk">
              <el-tooltip
                  :content="JUDGE_STATUS[row.myStatus]['name']"
                  placement="top"
              >
                <template v-if="row.myStatus == 0">
                  <i
                      :style="getIconColor(row.myStatus)"
                      class="el-icon-check"
                  ></i>
                </template>

                <template v-else-if="row.myStatus != -10">
                  <i
                      :style="getIconColor(row.myStatus)"
                      class="el-icon-minus"
                  ></i>
                </template>
              </el-tooltip>
            </template>
          </template>
        </vxe-table-column>
        <vxe-table-column
            :title="$t('m.Problem_ID')"
            field="problemId"
            show-overflow
            width="150"
        >
        <template v-slot="{ row }">
          <a class="title-a" @click="getProblemUri(row.id)">{{
              row.id
            }}</a>
        </template>
      </vxe-table-column>

        <vxe-table-column
            title="题目标题"
            field="title"
            min-width="150"
            show-overflow
        >
          <template v-slot="{ row }">
            <a class="title-a" @click="getProblemUri(row.id)">{{
                row.title
              }}</a>
          </template>
        </vxe-table-column>
      </vxe-table>
      </el-tab-pane>

        <!--提交-->
        <el-tab-pane :disabled="contestMenuDisabled" lazy name="ContestSubmissionList">
          <span slot="label"><i class="el-icon-menu"></i>&nbsp;{{ $t('m.Status') }}</span>
          <transition name="el-collapse-transition">
            <router-view
                v-if="route_name === 'ContestSubmissionList'"
            ></router-view>
          </transition>
        </el-tab-pane>

        <!--rank-->
        <!-- <el-tab-pane :disabled="contestMenuDisabled" lazy name="ContestRank">
          <span slot="label"
          ><i aria-hidden="true" class="fa fa-bar-chart"></i>&nbsp;{{
              $t('m.NavBar_Rank')
            }}</span
          >
          <transition name="el-collapse-transition">
            <router-view v-if="route_name === 'ContestRank'"></router-view>
          </transition>
        </el-tab-pane> -->

        <!--公告-->
        <!-- <el-tab-pane :disabled="contestMenuDisabled" lazy name="ContestAnnouncementList">
          <span slot="label"
          ><i aria-hidden="true" class="fa fa-bullhorn"></i>&nbsp;{{
              $t('m.Announcement')
            }}</span
          >
          <transition name="el-collapse-transition">
            <router-view
                v-if="route_name === 'ContestAnnouncementList'"
            ></router-view>
          </transition>
        </el-tab-pane> -->

        <!--讨论-->
        <!-- <el-tab-pane :disabled="contestMenuDisabled" lazy name="ContestComment">
          <span slot="label"><i aria-hidden="true" class="fa fa-commenting"></i>&nbsp;{{ $t('m.Comment') }}</span>
          <transition name="el-collapse-transition">
            <router-view v-if="route_name === 'ContestComment'"></router-view>
          </transition>
        </el-tab-pane> -->

        <!--打印-->
        <!-- <el-tab-pane v-if="contest.openPrint" :disabled="contestMenuDisabled" lazy name="ContestPrint">
          <span slot="label"><i class="el-icon-printer"></i>&nbsp;{{ $t('m.Print') }}</span>
          <transition name="el-collapse-transition">
            <router-view v-if="route_name === 'ContestPrint'"></router-view>
          </transition>
        </el-tab-pane> -->

        <!--admin helper-->
        <!-- <el-tab-pane v-if="showAdminHelper" :disabled="contestMenuDisabled" lazy name="ContestACInfo">
          <span slot="label"><i aria-hidden="true" class="el-icon-s-help"></i>&nbsp;{{ $t('m.Admin_Helper') }}</span>
          <transition name="el-collapse-transition">
            <router-view v-if="route_name === 'ContestACInfo'"></router-view>
          </transition>
        </el-tab-pane> -->

        <!--admin打印-->
        <!-- <el-tab-pane v-if="isSuperAdmin && contest.openPrint" :disabled="contestMenuDisabled" lazy
                     name="ContestAdminPrint">
          <span slot="label"><i class="el-icon-printer"></i>&nbsp;{{ $t('m.Admin_Print') }}</span>
          <transition name="el-collapse-transition">
            <router-view
                v-if="route_name === 'ContestAdminPrint'"
            ></router-view>
          </transition>
        </el-tab-pane> -->

        <!--重测-->
        <!-- <el-tab-pane v-if="isSuperAdmin" :disabled="contestMenuDisabled" lazy name="ContestRejudgeAdmin">
          <span slot="label"
          ><i aria-hidden="true" class="el-icon-refresh"></i>&nbsp;{{
              $t('m.Rejudge')
            }}</span
          >
          <transition name="el-collapse-transition">
            <router-view
                v-if="route_name === 'ContestRejudgeAdmin'"
            ></router-view>
          </transition>
        </el-tab-pane> -->
      </el-tabs>
    </div>
  </div>
</template>
<script>
import time from '@/common/time';
import moment from 'moment';
import api from '@/common/api';
import {mapActions, mapGetters, mapState} from 'vuex';
import {addCodeBtn} from '@/common/codeblock';
import {
  buildContestAnnounceKey,
  CONTEST_STATUS,
  CONTEST_STATUS_REVERSE,
  CONTEST_TYPE_REVERSE,
  RULE_TYPE,
} from '@/common/constants';
import myMessage from '@/common/message';
import storage from '@/common/storage';

export default {
  name: '',
  data() {
    return {
      route_name: 'contestDetails',
      timer: null,
      CONTEST_STATUS: {},
      CONTEST_STATUS_REVERSE: {},
      CONTEST_TYPE_REVERSE: {},
      RULE_TYPE: {},
      Contest:{},
      problemList:[],
      // btnLoading: false,
      // contestPassword: '',
    };
  },
  created() {
    this.contestID = this.$route.params.contestID;
    this.route_name = this.$route.name;
    // this.route_name = 'ContestProblemList';
    if (this.route_name == 'ContestProblemDetails') {
      this.route_name = 'ContestProblemList';
    }
    if (this.route_name == 'ContestSubmissionDetail') {
      this.route_name = 'ContestSubmissionList';
    }
    this.CONTEST_TYPE_REVERSE = Object.assign({}, CONTEST_TYPE_REVERSE);
    this.CONTEST_STATUS = Object.assign({}, CONTEST_STATUS);
    this.CONTEST_STATUS_REVERSE = Object.assign({}, CONTEST_STATUS_REVERSE);
    this.RULE_TYPE = Object.assign({}, RULE_TYPE);
   ;
    api.getContest(this.contestID).then(
      (res) => {
           this.Contest=res.data;
           let problem_list = this.Contest.problem_ids;
          for( let i=0;i<problem_list.length;i++){
            let qId=problem_list[i];
            // console.log(qi)
            api.getProblem(qId).then(
              (res) => {
                console.log("success");
                console.log("Res",res);
                let result={};
                result= res.data;
                this.problemList.push(result);
              },
              (err) => {
                console.log("fail");
                this.loading = false;
              });
            }
          },
          (err) => {
            console.log("fail");
            this.loading = false;
          });
    console.log("problem",problem_list)
   


  },
  methods: {
    // ...mapActions(['changeDomTitle']),
    // formatTooltip(val) {
    //   if (this.contest.status == -1) {
    //     // 还未开始
    //     return '00:00:00';
    //   } else if (this.contest.status == 0) {
    //     return time.secondFormat(this.BeginToNowDuration); // 格式化时间
    //   } else {
    //     return time.secondFormat(this.contest.duration);
    //   }
    // },
    // checkPassword() {
    //   if (this.contestPassword === '') {
    //     myMessage.warning(this.$i18n.t('m.Enter_the_contest_password'));
    //     return;
    //   }
    //   this.btnLoading = true;
    //   api.registerContest(this.contestID + '', this.contestPassword).then(
    //       (res) => {
    //         myMessage.success(this.$i18n.t('m.Register_contest_successfully'));
    //         this.$store.commit('contestIntoAccess', {intoAccess: true});
    //         this.btnLoading = false;
    //       },
    //       (res) => {
    //         this.btnLoading = false;
    //       }
    //   );
    // },
    tabClick(tab) {
      let name = tab.name;
      if (name !== this.$route.name) {
        this.$router.push({name: name});
      }
    },
   getProblemUri(problemId) {
      this.$router.push({
        name: 'ProblemDetails',
        params: {
          problemID: problemId,
          contestID:this.contest.id,
        },
      });
    },
  },
  computed: {
    ...mapState({
      contest: (state) => state.contest.contest,
      now: (state) => state.contest.now,
    }),
    ...mapGetters([
      'contestMenuDisabled',
      'contestRuleType',
      'contestStatus',
      'countdown',
      'BeginToNowDuration',
      'isContestAdmin',
      'isSuperAdmin',
      'ContestRealTimePermission',
      'passwordFormVisible',
      'userInfo',
    ]),
    progressValue: {
      get: function () {
        return this.$store.getters.progressValue;
      },
      set: function () {
      },
    },
    timeStep() {
      // 时间段平分滑条长度
      return 100 / this.contest.duration;
    },
    countdownColor() {
      if (this.contestStatus) {
        return 'color:' + CONTEST_STATUS_REVERSE[this.contestStatus].color;
      }
    },
    showAdminHelper() {
      return this.isContestAdmin && this.contestRuleType === RULE_TYPE.ACM;
    },
    descriptionHtml() {
      if (this.contest.description) {
        return this.$markDown.render(this.contest.description);
      }
    },
    contestEnded() {
      return this.contestStatus === CONTEST_STATUS.ENDED;
    },
  },
  watch: {
    $route(newVal) {
      this.route_name = newVal.name;
      if (newVal.name == 'ContestProblemDetails') {
        this.route_name = 'ContestProblemList';
      }
      if (this.route_name == 'ContestSubmissionDetail') {
        this.route_name = 'ContestSubmissionList';
      }
      this.contestID = newVal.params.contestID;
      // this.changeDomTitle({title: this.contest.title});
    },
  },
  beforeDestroy() {
    clearInterval(this.timer);
    clearInterval(this.announceTimer);
    this.$store.commit('clearContest');
  },
};
</script>
<style scoped>
.panel-title {
  font-size: 1.5rem !important;
  font-weight: 500;
}

@media screen and (min-width: 768px) {
  .contest-time .left {
    text-align: left;
  }

  .contest-time .right {
    text-align: right;
  }

  .password-form-card {
    width: 400px;
    margin: 0 auto;
  }
}

@media screen and (max-width: 768px) {
  .contest-time .left,
  .contest-time .right {
    text-align: center;
  }
}

/deep/ .el-slider__button {
  width: 20px !important;
  height: 20px !important;
  background-color: #409eff !important;
}

/deep/ .el-slider__button-wrapper {
  z-index: 500;
}

/deep/ .el-slider__bar {
  height: 10px !important;
  background-color: #09be24 !important;
}

/deep/ .el-card__header {
  border-bottom: 0px;
  padding-bottom: 0px;
}

/deep/ .el-tabs__nav-wrap {
  background: #fff;
  border-radius: 3px;
}

/deep/ .el-tabs--top .el-tabs__item.is-top:nth-child(2) {
  padding-left: 20px;
}

.contest-title {
  text-align: center;
}

.contest-time {
  width: 100%;
  font-size: 16px;
}

.el-tag--dark {
  border-color: #fff;
}

.el-tag {
  color: rgb(25, 190, 107);
  background: #fff;
  border: 1px solid #e9eaec;
  font-size: 18px;
}

.sub-menu {
  margin-top: 15px;
}

.password-form-tips {
  text-align: center;
  font-size: 14px;
}
</style>
