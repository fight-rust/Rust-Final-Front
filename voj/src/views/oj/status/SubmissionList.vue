<template>
  <el-row class="flex-container">
    <div id="main">
      <el-card shadow>
        <div slot="header">
          <el-row :gutter="18">
            <el-col :lg="2" :md="4">
              <span class="panel-title hidden-md-and-down">{{
                  $t('m.List')
                }}</span>
            </el-col>
            <el-col :lg="4" :md="4" :sm="8" :xs="10">
              <el-switch
                  v-model="formFilter.onlyMine"
                  :active-text="$t('m.Mine')"
                  :inactive-text="$t('m.All')"
                  :width="40"
                  style="display: block"
                  @change="handleOnlyMine"
              >
              </el-switch>
            </el-col>

            <!-- <el-col :lg="4" :md="5" :sm="8" :xs="10" style="padding-top: 5px;">
              <el-dropdown
                  class="drop-menu"
                  placement="bottom"
                  trigger="hover"
                  @command="handleStatusChange"
              >
                <span class="el-dropdown-link">
                  {{ status }}
                  <i class="el-icon-caret-bottom"></i>
                </span>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="All">{{
                      $t('m.All')
                    }}
                  </el-dropdown-item> -->
                  <!-- <el-dropdown-item
                      v-for="result in Object.keys(JUDGE_STATUS_LIST)"
                      :key="result"
                      :command="result"
                  >
                    {{ JUDGE_STATUS_LIST[result].name }}
                  </el-dropdown-item> -->
                  <!-- <el-dropdown-item
                  v-for="result in submissions"
                  :key="result"
                  :command="result"
                   >
                {{result.id}}
              </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </el-col> -->

            <el-col :lg="4" :md="5" :sm="8" class="hidden-xs-only">
              <el-button
                  icon="el-icon-refresh"
                  round
                  size="small"
                  type="primary"
                  @click="getUpdatedSubmissions"
              >{{ $t('m.Refresh') }}
              </el-button
              >
            </el-col>
            <el-col :xs="4" class="hidden-sm-and-up">
              <el-button
                  circle
                  icon="el-icon-refresh"
                  size="small"
                  type="primary"
                  @click="getUpdatedSubmissions"
              ></el-button>
            </el-col>

            <!-- <el-col :lg="5" :md="5" :sm="12" :xs="24" class="search">
              <vxe-input
                  v-model="formFilter.problemID"
                  :placeholder="$t('m.Enter_Problem_ID')"
                  size="medium"
                  type="search"
                  @keyup.enter.native="handleQueryChange('problemID')"
                  @search-click="handleQueryChange('problemID')"
              ></vxe-input>
            </el-col>
            <el-col :lg="5" :md="5" :sm="12" :xs="24" class="search">
              <vxe-input
                  v-model="formFilter.username"
                  :disabled="formFilter.onlyMine"
                  :placeholder="$t('m.Enter_Author')"
                  size="medium"
                  type="search"
                  @keyup.enter.native="handleQueryChange('username')"
                  @search-click="handleQueryChange('username')"
              ></vxe-input>
            </el-col> -->
          </el-row>
        </div>
        <el-table
        :data="submissions"
        style="width: 100%" stripe>
        <el-table-column
          prop="id"
          label="????????????"
          width="150">
        </el-table-column>
        <el-table-column
          prop="contest"
          label="????????????"
          width="150">
        </el-table-column>
        <el-table-column
          prop="problem"
          label="????????????"
          width="150">
        </el-table-column>
          <el-table-column
          prop="user"
          label="??????"
          width="180">
        </el-table-column>
          <el-table-column
          prop="created_time"
          label="????????????">
        </el-table-column>
          <el-table-column
          prop="result"
          label="????????????">
        </el-table-column>
        <el-table-column
        prop="run_time"
        label="????????????">
    </el-table-column>
      </el-table>
      </el-card>
      <Pagination
          :current.sync="currentPage"
          :layout="'prev, pager, next, sizes'"
          :page-size="limit"
          :total="total"
          @on-change="changeRoute"
          @on-page-size-change="onPageSizeChange"
      ></Pagination>
    </div>
  </el-row>
</template>

<script>
import {mapActions, mapGetters} from 'vuex';
import api from '@/common/api';
import {CONTEST_STATUS, JUDGE_STATUS, JUDGE_STATUS_RESERVE, RULE_TYPE,} from '@/common/constants';
import utils from '@/common/utils';
import Pagination from '@/components/oj/common/Pagination';
import myMessage from '@/common/message';
import 'element-ui/lib/theme-chalk/display.css';

export default {
  name: 'submissionList',
  components: {
    Pagination,
  },
  data() {
    return {
      formFilter: {
        onlyMine: false,
        status: '',
        username: '',
        problemID: '',
      },
      loadingTable: false,
      submissions: [],
      needCheckSubmitIds: {}, // ???????????????6???7???????????????Id ????????????????????????
      total: 30,
      limit: 15,
      currentPage: 1,
      contestID: '',
      routeName: '',
      checkStatusNum: 0,
      JUDGE_STATUS: '',
      JUDGE_STATUS_LIST: '',
      autoCheckOpen: false,
      JUDGE_STATUS_RESERVE: {},
      CONTEST_STATUS: {},
      RULE_TYPE: {},
      subList:{},
      filterSubmissions:[],
    };
  },
  created() {
    this.init();
  },
  mounted() {
    this.JUDGE_STATUS = Object.assign({}, JUDGE_STATUS);
    this.JUDGE_STATUS_LIST = Object.assign({}, JUDGE_STATUS);
    this.JUDGE_STATUS_RESERVE = Object.assign({}, JUDGE_STATUS_RESERVE);
    this.CONTEST_STATUS = Object.assign({}, CONTEST_STATUS);
    this.RULE_TYPE = Object.assign({}, RULE_TYPE);
    // ???????????????????????????Compiling,Judging,Submitting,Not Submitted,Submitted Unknown Result ????????????
    delete this.JUDGE_STATUS_LIST['6'];
    delete this.JUDGE_STATUS_LIST['7'];
    delete this.JUDGE_STATUS_LIST['9'];
    delete this.JUDGE_STATUS_LIST['-5'];
    delete this.JUDGE_STATUS_LIST['-10'];
    this.getData();
  },
  methods: {
    init() {
      this.checkStatusNum = 0;
      this.contestID = this.$route.params.contestID;
      let query = this.$route.query;
      this.formFilter.problemID = query.problemID;
      this.formFilter.username = query.username || '';
      this.formFilter.onlyMine = query.onlyMine + '' == 'true'; // ???????????????????????????
      this.formFilter.status = query.status;
      this.formFilter.completeProblemID = query.completeProblemID || false;
      if (this.formFilter.onlyMine) {
        // ?????????????????????????????? ?????????????????????????????????
        this.formFilter.username = '';
      }
      this.currentPage = parseInt(query.currentPage) || 1;
      if (this.currentPage < 1) {
        this.currentPage = 1;
      }
      this.limit = parseInt(query.limit) || 15;
      this.routeName = this.$route.name;
    },

    getData() {
      this.getSubmissions();
    },

    buildQuery() {
      return {
        onlyMine: this.formFilter.onlyMine,
        status: this.formFilter.status,
        username: this.formFilter.username,
        problemID: this.formFilter.problemID,
        currentPage: this.currentPage,
        limit: this.limit,
        completeProblemID: this.formFilter.completeProblemID,
      };
    },

    submissionTimeFormat(time) {
      return utils.submissionTimeFormat(time);
    },

    submissionMemoryFormat(memory) {
      return utils.submissionMemoryFormat(memory);
    },

    submissionLengthFormat(length) {
      return utils.submissionLengthFormat(length);
    },
    reSubmit(row) {
      api.reSubmitRemoteJudge(row.submitId).then((res) => {
        let xTable = this.$refs.xTable;
        // ???????????????????????????????????????????????????????????????
        row.status = res.data.data.status;
        row.time = res.data.data.time;
        row.memory = res.data.data.memory;
        row.errorMessage = res.data.data.errorMessage;
        row.judger = res.data.data.judger;
        // ???????????????????????????view
        xTable.reloadRow(row, null, null);

        this.submissions[row.index] = res.data.data;
        myMessage.success(res.data.msg);

        // ?????????????????????
        this.needCheckSubmitIds[row.submitId] = row.index;

        this.checkStatusNum = 0;
        if (!this.autoCheckOpen) {
          // ????????????????????????????????????????????????????????????????????????
          this.checkSubmissionsStatus();
        }
      });
    },
    getSubmissions() {
      let params = this.buildQuery();
      params.contestID = this.contestID;
      if (this.contestID) {
        if (this.contestStatus == CONTEST_STATUS.SCHEDULED) {
          params.beforeContestSubmit = true;
        } else {
          params.beforeContestSubmit = false;
        }
      }
      // if (this.formFilter.onlyMine) {
      //   // ?????????????????????????????????
      //   // if (this.isAuthenticated) {
      //   //   params.username = ''; // ??????????????????????????????????????????????????????????????????????????????
      //   //   this.formFilter.username = '';
      //   // } else {
      //     if (this.$store.getters.userName === 'username') {
      //       // console.log('?????????');
      //       this.formFilter.onlyMine = false;
      //       myMessage.error(this.$i18n.t('m.Please_login_first'));
      //       return;
      //     } else {
      //       params.username = ''; // ??????????????????????????????????????????????????????????????????????????????
      //       this.formFilter.username = '';
      //     }
      // }

      this.loadingTable = true;
      this.submissions = [];
      this.needCheckSubmitIds = {};
      // let func = this.contestID
      //     ? 'getContestSubmissionList'
      //     : 'getSubmissionList';
      // api[func](this.limit, utils.filterEmptyValue(params))
      //     .then((res) => {
      //       let data = res.data.data;
      //       let index = 0;
      //       for (let v of data.records) {
      //         if (
      //             v.status == JUDGE_STATUS_RESERVE['Pending'] ||
      //             v.status == JUDGE_STATUS_RESERVE['Compiling'] ||
      //             v.status == JUDGE_STATUS_RESERVE['Judging']
      //         ) {
      //           this.needCheckSubmitIds[v.submitId] = index;
      //         }
      //         v.loading = false;
      //         v.index = index;
      //         index += 1;
      //       }
      //       this.loadingTable = false;
      //       this.submissions = data.records;
      //       this.total = data.total;
      //       if (Object.keys(this.needCheckSubmitIds).length > 0) {
      //         this.checkSubmissionsStatus();
      //       }
      //     })
      //     .catch(() => {
      //       this.loadingTable = false;
      //     });

      api.getSubmissionList().then(
          (res) => {
            console.log("success",res.data);
            this.submissions = res.data;
            console.log(this.submissions);
            this.submissions.forEach(sub=>{
              if(sub.result=='Compiling'){
                sub.run_time = '/';
              }
              var date = new Date(sub.created_time).toJSON();
              sub.created_time=new Date(+new Date(date)+8*3600*1000).toISOString().replace(/T/g,' ').replace(/\.[\d]{3}Z/,'');
            })
            // this.subList = {};
            // this.total = res.data.data.total;
            this.loadingTable = false;
            // for(sub in this.submissions){
            //   Object.assign(this.subList,sub);
            // }
            // console.log(this.subList);
          },
          (err) => {
            console.log("fail",err);
            this.loadingTable = false;
          }
      );
    },
    // ????????????????????? ?????????Pending???6??????Judging???7?????????????????????2???????????????????????????
    checkSubmissionsStatus() {
      // ??????setTimeout??????????????????
      if (this.refreshStatus) {
        // ????????????????????????????????????????????????, ?????????, ??????????????????timeout???????????????????????????
        clearTimeout(this.refreshStatus);
        this.autoCheckOpen = false;
      }
      const checkStatus = () => {
        let submitIds = this.needCheckSubmitIds;
        let func = this.contestID
            ? 'checkContestSubmissionsStatus'
            : 'checkSubmissionsStatus';
        api[func](Object.keys(submitIds), this.contestID).then(
            (res) => {
              let result = res.data.data;
              if (!this.$refs.xTable) {
                // ????????????????????????view??????
                return;
              }
              let viewData = this.$refs.xTable.getTableData().tableData;
              for (let key in submitIds) {
                let submitId = parseInt(key);
                if (!result[submitId]) {
                  continue;
                }
                // ??????????????????
                this.submissions[submitIds[key]] = result[submitId];
                // ??????view???????????????f??????????????????????????????????????????ip
                viewData[submitIds[key]].status = result[submitId].status;
                viewData[submitIds[key]].score = result[submitId].score;
                viewData[submitIds[key]].time = result[submitId].time;
                viewData[submitIds[key]].memory = result[submitId].memory;
                viewData[submitIds[key]].judger = result[submitId].judger;
                // ???????????????????????????view???
                this.$refs.xTable.reloadRow(viewData[submitIds[key]], null, null);

                if (
                    result[submitId].status != JUDGE_STATUS_RESERVE['Pending'] &&
                    result[submitId].status != JUDGE_STATUS_RESERVE['Compiling'] &&
                    result[submitId].status != JUDGE_STATUS_RESERVE['Judging']
                ) {
                  delete this.needCheckSubmitIds[key];
                }
              }
              // ????????????????????????????????????????????????????????????60s???2s*30??????????????????????????????????????????????????????????????????????????????????????????????????????
              if (
                  Object.keys(this.needCheckSubmitIds).length == 0 ||
                  this.checkStatusNum == 30
              ) {
                clearTimeout(this.refreshStatus);
                this.autoCheckOpen = false;
              } else {
                this.checkStatusNum += 1;
                console.log(this.checkStatusNum)
                this.refreshStatus = setTimeout(checkStatus, 2000);
              }
            },
            (res) => {
              clearTimeout(this.refreshStatus);
              this.autoCheckOpen = false;
            }
        );
      };
      // ?????????2???????????????????????????
      this.checkStatusNum += 1;
      this.refreshStatus = setTimeout(checkStatus, 2000);
      this.autoCheckOpen = true;
    },
    onPageSizeChange(pageSize) {
      this.limit = pageSize;
      this.changeRoute();
    },
    // ??????route??? ????????????route???????????????????????????????????????route history??? ??????????????????????????????????????????
    changeRoute() {
      let query = this.buildQuery();
      query.contestID = this.contestID;
      let queryParams = utils.filterEmptyValue(query);
      // ??????????????????????????????????????????????????????????????????????????????????????????????????????
      let equal = true;
      for (let key in queryParams) {
        if (queryParams[key] != this.$route.query[key]) {
          equal = false;
          break;
        }
      }
      if (equal) {
        // ???????????????????????????
        if (
            Object.keys(queryParams).length !=
            Object.keys(this.$route.query).length
        ) {
          equal = false;
        }
      }

      if (!equal) {
        // ??????????????????????????????????????????
        let routeName = queryParams.contestID
            ? 'ContestSubmissionList'
            : 'SubmissionList';
        this.$router.push({
          name: routeName,
          query: queryParams,
        });
      }
    },
    goRoute(route) {
      this.$router.push(route);
    },
    goUserHome(username, uid) {
      this.$router.push({
        path: '/user-home',
        query: {uid, username},
      });
    },
    handleStatusChange(status) {
      if (status == 'All') {
        this.formFilter.status = '';
      } else {
        this.formFilter.status = status;
      }
      this.currentPage = 1;
      this.changeRoute();
    },
    handleQueryChange(searchParam) {
      if (searchParam == 'problemID') {
        this.formFilter.completeProblemID = false; // ?????????????????????displayID???
      }
      this.currentPage = 1;
      this.changeRoute();
    },
    handleRejudge(row) {
      this.submissions[row.index].loading = true;
      api.submissionRejudge(row.submitId).then(
          (res) => {
            let xTable = this.$refs.xTable;
            // ?????????????????????????????????????????????????????????
            row.status = res.data.data.status;
            row.score = null;
            row.time = res.data.data.time;
            row.memory = res.data.data.memory;
            row.errorMessage = res.data.data.errorMessage;
            row.judger = res.data.data.judger;
            row.loading = false;
            // ???????????????????????????view
            xTable.reloadRow(row, null, null);

            this.submissions[row.index] = res.data.data;
            this.submissions[row.index].loading = false;
            myMessage.success(this.$i18n.t('m.Rejudge_successfully'));

            // ?????????????????????
            this.needCheckSubmitIds[row.submitId] = row.index;
            this.checkStatusNum = 0;
            if (!this.autoCheckOpen) {
              // ????????????????????????????????????????????????????????????????????????
              this.checkSubmissionsStatus();
            }
          },
          () => {
            this.submissions[row.index].loading = false;
          }
      );
    },

    handleOnlyMine() {
      if (this.formFilter.onlyMine) {
        // ?????????????????????????????????
        // if (this.isAuthenticated) {
          console.log('---',this.$store.getters.userName);
        if(this.$store.getters.userName != 'username'){
          this.formFilter.username = '';
          let data = {
            user_name : this.$store.getters.userName,
          };
          api.getFilteredSubmit(data).then(
              (res) => {
                console.log("success",res.data);
                this.submissions = res.data;
                console.log(this.submissions);
                this.submissions.forEach(sub=>{
                  var date = new Date(sub.created_time).toJSON();
                  sub.created_time=new Date(+new Date(date)+8*3600*1000).toISOString().replace(/T/g,' ').replace(/\.[\d]{3}Z/,'');
                })
              },
              (err) => {
                console.log("fail",err);
                // this.loadingTable = false;
              }
          );
        } 
        else {
          this.formFilter.onlyMine = false;
          myMessage.error(this.$i18n.t('m.Please_login_first'));
          return;
        }
      }
      else{
        this.getSubmissions();
      }
      this.currentPage = 1;
      // this.changeRoute();
    },

    getUpdatedSubmissions(){
      if (this.formFilter.onlyMine) {
        // ?????????????????????????????????
        // if (this.isAuthenticated) {
          console.log('---',this.$store.getters.userName);
        if(this.$store.getters.userName != 'username'){
          this.formFilter.username = '';
          let data = {
            user_name : this.$store.getters.userName,
          };
          api.getFilteredSubmit(data).then(
              (res) => {
                console.log("success",res.data);
                this.submissions = res.data;
                console.log(this.submissions);
                this.submissions.forEach(sub=>{
                  if(sub.result=='Compiling'){
                      sub.run_time = '/';
                    }
                  var date = new Date(sub.created_time).toJSON();
                  sub.created_time=new Date(+new Date(date)+8*3600*1000).toISOString().replace(/T/g,' ').replace(/\.[\d]{3}Z/,'');
                })
              },
              (err) => {
                console.log("fail",err);
                // this.loadingTable = false;
              }
          );
        } 
        else {
          this.formFilter.onlyMine = false;
          myMessage.error(this.$i18n.t('m.Please_login_first'));
          return;
        }
      }
      else{
        this.getSubmissions();
      }
      this.currentPage = 1;
    },
    ...mapActions(['changeModalStatus']),

    showSubmitDetail(row) {
      if (row.cid != 0) {
        // ??????????????????
        this.$router.push({
          name: 'ContestSubmissionDetail',
          params: {
            contestID: this.$route.params.contestID,
            problemID: row.displayId,
            submitID: row.submitId,
          },
        });
      } else {
        this.$router.push({
          name: 'SubmissionDetail',
          params: {submitID: row.submitId},
        });
      }
    },
    getProblemUri(pid, isContest) {
      if (isContest) {
        this.$router.push({
          name: 'ContestProblemDetails',
          params: {
            contestID: this.$route.params.contestID,
            problemID: pid,
          },
        });
      } else {
        this.$router.push({
          name: 'ProblemDetails',
          params: {
            problemID: pid,
          },
        });
      }
    },
    getStatusColor(status) {
      console.log(status)
      return 'el-tag el-tag--medium status-' + JUDGE_STATUS[status]['color'];
    },
    tableRowClassName({row, rowIndex}) {
      if (row.username == this.userInfo.username && this.isAuthenticated) {
        return 'own-submit-row';
      }
    }
  },
  computed: {
    filterSubmissions()
    {
      console.log(this.$store.getters.userName);
      console.log('123',this.submissions);
      return this.submissions.filter(sub => !this.formFilter.onlyMine || sub.user.toLowerCase().includes(this.search.toLowerCase()));
      // return this.submissions.filter((sub) => {return true});
    },
    ...mapGetters([
      'isAuthenticated',
      'userInfo',
      'isSuperAdmin',
      'isAdminRole',
      'contestRuleType',
      'contestStatus',
      'ContestRealTimePermission',
    ]),
    title() {
      if (!this.contestID) {
        return 'Status';
      } else if (this.problemID) {
        return 'Problem Submissions';
      } else {
        return 'Submissions';
      }
    },
    status() {
      return this.formFilter.status === ''
          ? this.$i18n.t('m.Status')
          : JUDGE_STATUS[this.formFilter.status]
              ? JUDGE_STATUS[this.formFilter.status].name
              : this.$i18n.t('m.Status');
    },
    scoreColumnVisible() {
      return (
          (this.contestID && this.contestRuleType == this.RULE_TYPE.OI) ||
          !this.contestID
      );
    },
  },
  watch: {
    $route(newVal, oldVal) {
      if (newVal !== oldVal) {
        if (this.autoCheckOpen) {
          clearInterval(this.refreshStatus);
        }
        this.init();
        this.getData();
      }
    },
    isAuthenticated() {
      this.init();
      this.getData();
    },
  },
  beforeRouteLeave(to, from, next) {
    // ???????????????????????????????????????
    clearInterval(this.refreshStatus);
    next();
  },
};
</script>

<style scoped>
@media only screen and (max-width: 767px) {
  .search {
    margin-top: 20px;
  }
}

.flex-container #main {
  flex: auto;
}

.flex-container .filter {
  margin-right: -10px;
}

.flex-container #contest-menu {
  flex: none;
  width: 210px;
}

/deep/ .el-card__header {
  border-bottom: 0px;
  padding-bottom: 0px;
  text-align: center;
}

/deep/ .el-dialog {
  border-radius: 6px !important;
  text-align: center;
}

/deep/ .el-switch {
  padding-top: 6px;
}

@media only screen and (min-width: 768px) and (max-width: 992px) {
  .el-col-sm-12 {
    padding-top: 10px;
  }
}

@media screen and (min-width: 1050px) {
  /deep/ .vxe-table--body-wrapper {
    overflow-x: hidden !important;
  }
}
</style>
