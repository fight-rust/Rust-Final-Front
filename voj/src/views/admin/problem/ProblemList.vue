<template>
  <div>
    <el-card>
      <div slot="header">
        <span class="panel-title home-title">{{
            query.contestId ? $t('m.Contest_Problem_List') : $t('m.Problem_List')
          }}</span>

      </div>
      <vxe-table
          ref="adminProblemList"
          :data="problemList"
          :loading="loading"
          align="center"
          auto-resize
          stripe
      >
        <vxe-table-column field="pid" min-width="64" title="题目ID"></vxe-table-column>
        <vxe-table-column field="ptitle" min-width="64" title="题目标题"></vxe-table-column>

      </vxe-table>
    </el-card>

    <el-dialog
        v-if="query.contestId"
        :title="$t('m.Add_Contest_Problem')"
        :visible.sync="addProblemDialogVisible"
        width="90%"
        :close-on-click-modal="false"
    >
      <AddPublicProblem
          :contestID="query.contestId"
          @on-change="getProblemList"
          @getTips="getContestProblemTips"
      ></AddPublicProblem>
    </el-dialog>

    <el-dialog
        :title="$t('m.Add_Remote_OJ_Problem')"
        :visible.sync="AddRemoteOJProblemDialogVisible"
        width="350px"
        :close-on-click-modal="false"
    >
      <el-form>
        <el-form-item :label="$t('m.Remote_OJ')">
          <el-select v-model="otherOJName" size="small">
            <el-option
                v-for="(remoteOj, index) in REMOTE_OJ"
                :key="index"
                :label="remoteOj.name"
                :value="remoteOj.key"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item :label="$t('m.Problem_ID')" required>
          <el-input v-model="otherOJProblemId" size="small"></el-input>
        </el-form-item>

<!--        <el-form-item-->
<!--            v-if="query.contestId"-->
<!--            :label="$t('m.Enter_The_Problem_Display_ID_in_the_Contest')"-->
<!--            required-->
<!--        >-->
<!--          <el-input v-model="displayId" size="small"></el-input>-->
<!--        </el-form-item>-->

        <el-form-item style="text-align:center">
          <el-button
              :loading="addRemoteOJProblemLoading"
              icon="el-icon-plus"
              type="primary"
              @click="addRemoteOJProblem"
          >{{ $t('m.Add') }}
          </el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import api from '@/common/api';
import utils from '@/common/utils';
import AddPublicProblem from '@/components/admin/AddPublicProblem.vue';
import myMessage from '@/common/message';
import {REMOTE_OJ} from '@/common/constants';
import {mapGetters} from 'vuex';

export default {
  name: 'ProblemList',
  components: {
    AddPublicProblem,
  },
  data() {
    return {
      total: 0,
      query: {
        problemListAuth: 0,
        oj: 'All',
        pageSize: 10,
        keyword: '',
        currentPage: 1,
        contestId: null,
      },
      problemList: [],
      contestProblemMap: {},
      loading: false,
      routeName: '',
      // for make public use
      currentProblemID: '',
      currentRow: {},
      addProblemDialogVisible: false,
      AddRemoteOJProblemDialogVisible: false,
      addRemoteOJProblemLoading: false,
      otherOJName: REMOTE_OJ[0].key,
      otherOJProblemId: '',
      REMOTE_OJ: {},
      displayId: '',
      contestProblemTips: '',
      showPagination: false,

      predefineColors: [
        '#ff4500',
        '#ff8c00',
        '#ffd700',
        '#90ee90',
        '#00ced1',
        '#1e90ff',
        '#c71585',
      ],
    };
  },
  mounted() {
    this.init();
  },
  computed: {
    ...mapGetters(['userInfo', 'isSuperAdmin', 'isProblemAdmin']),
    isContest() {
      return !(this.routeName == 'admin-problem-list' && !this.query.contestId);
    },
  },
  methods: {
    init() {
      this.routeName = this.$route.name;
      let query = this.$route.query;
      this.query.currentPage = parseInt(query.currentPage) || 1;
      this.query.pageSize = parseInt(query.pageSize) || 10;
      this.query.keyword = query.keyword;
      this.query.problemListAuth = query.problemListAuth
          ? parseInt(query.problemListAuth)
          : 0;
      this.query.oj = query.oj || 'All';
      this.query.contestId = this.$route.params.contestId;
      this.contestProblemMap = {};
      this.getProblemList();
      this.REMOTE_OJ = Object.assign({}, REMOTE_OJ);
    },
    getContestProblemTips(data){
      this.contestProblemTips = data
    },
    goEdit(problemId) {
      if (this.routeName === 'admin-problem-list') {
        this.$router.push({
          name: 'admin-edit-problem',
          params: {problemId},
          query: {
            back: this.$route.fullPath,
          },
        });
      } else if (this.routeName === 'admin-contest-problem-list') {
        this.$router.push({
          name: 'admin-edit-contest-problem',
          params: {problemId: problemId, contestId: this.query.contestId},
        });
      }
    },
    goCreateProblem() {
      if (this.routeName === 'admin-problem-list') {
        this.$router.push({
          name: 'admin-create-problem',
          query: {
            back: this.$route.fullPath,
          },
        });
      } else if (this.routeName === 'admin-contest-problem-list') {
        this.$router.push({
          name: 'admin-create-contest-problem',
          params: {contestId: this.query.contestId},
        });
      }
    },

    pushRouter() {
      if (this.query.contestId) {
        this.$router.push({
          name: 'admin-contest-problem-list',
          query: this.query,
          params: {
            contestId: this.query.contestId,
          },
        });
      } else {
        this.$router.push({
          name: 'admin-problem-list',
          query: this.query,
        });
      }
    },

    // 切换页码回调
    currentChange(page) {
      this.query.currentPage = page;
      this.pushRouter();
    },
    onPageSizeChange(pageSize) {
      this.query.pageSize = pageSize;
      this.pushRouter();
    },
    getProblemList() {
      let params = {
        limit: this.query.pageSize,
        currentPage: this.query.currentPage,
        keyword: this.query.keyword,
        cid: this.query.contestId,
        oj: this.query.oj,
      };
      if (this.problemListAuth != 0) {
        params['auth'] = this.query.problemListAuth;
      }
      this.loading = true;
      if (this.routeName === 'admin-problem-list') {
        api.admin_getProblemList(params).then(
            (res) => {
              for(let i=0;i<res.data.pid.length;i++){
                let temp={pid:res.data.pid[i],ptitle:res.data.ptitle[i]};
                this.problemList.push(temp);
              }
              this.loading = false;
              this.showPagination = true;
              this.total = res.data.data.total;
              this.problemList = res.data.data.records;
              this.contestProblemMap = {};
            },
            (err) => {
              this.loading = false;
              this.showPagination = true;
            }
        );
      } else {
        api.admin_getContestProblemList(params).then(
            (res) => {

              this.loading = false;
              this.showPagination = false;
              this.total = res.data.data.problemList.total;
              this.problemList = res.data.data.problemList.records;
              this.contestProblemMap = res.data.data.contestProblemMap;
            },
            (err) => {
              this.loading = false;
              this.showPagination = false;
            }
        );
      }
    },

    changeProblemAuth(row) {
      api.admin_changeProblemAuth(row).then((res) => {
        myMessage.success(this.$i18n.t('m.Update_Successfully'));
      });
    },

    deleteProblem(id) {
      this.$confirm(this.$i18n.t('m.Delete_Problem_Tips'), 'Tips', {
        type: 'warning',
      }).then(
          () => {
            let funcName =
                this.routeName === 'admin-problem-list'
                    ? 'admin_deleteProblem'
                    : 'admin_deleteContestProblem';
            api[funcName](id, null)
                .then((res) => {
                  myMessage.success(this.$i18n.t('m.Delete_successfully'));
                  this.getProblemList();
                })
                .catch(() => {
                });
          },
          () => {
          }
      );
    },
    removeProblem(pid) {
      this.$confirm(this.$i18n.t('m.Remove_Contest_Problem_Tips'), 'Tips', {
        type: 'warning',
      }).then(
          () => {
            api
                .admin_deleteContestProblem(pid, this.query.contestId)
                .then((res) => {
                  myMessage.success('success');
                  this.getProblemList();
                })
                .catch(() => {
                });
          },
          () => {
          }
      );
    },
    updateProblem(row) {
      let data = Object.assign({}, row);
      let funcName = '';
      if (this.query.contestId) {
        data.contest_id = this.query.contestId;
        funcName = 'admin_editContestProblem';
      } else {
        funcName = 'admin_editProblem';
      }
      api[funcName](data)
          .then((res) => {
            myMessage.success(this.$i18n.t('m.Update_Successfully'));
            this.getProblemList();
          })
          .catch(() => {
          });
    },
    downloadTestCase(problemID) {
      let url = '/api/file/download-testcase?pid=' + problemID;
      utils.downloadFile(url).then(() => {
        this.$alert(this.$i18n.t('m.Download_Testcase_Success'), 'Tips');
      });
    },
    ProblemListChangeFilter() {
      this.pushRouter();
    },
    filterByKeyword() {
      this.query.currentPage = 1;
      this.pushRouter();
    },
    addRemoteOJProblem() {
      if (!this.otherOJProblemId) {
        myMessage.error(this.$i18n.t('m.Problem_ID_is_required'));
        return;
      }

      // if (!this.displayId && this.query.contestId) {
      //   myMessage.error(
      //       this.$i18n.t('m.The_Problem_Display_ID_in_the_Contest_is_required')
      //   );
      //   return;
      // }

      this.addRemoteOJProblemLoading = true;
      let funcName = '';
      if (this.query.contestId) {
        funcName = 'admin_addContestRemoteOJProblem';
      } else {
        funcName = 'admin_addRemoteOJProblem';
      }
      api[funcName](
          this.otherOJName,
          this.otherOJProblemId,
          this.query.contestId,
          this.displayId
      ).then(
          (res) => {
            this.addRemoteOJProblemLoading = false;
            this.AddRemoteOJProblemDialogVisible = false;
            myMessage.success(this.$i18n.t('m.Add_Successfully'));
            this.currentChange(1);
          },
          (err) => {
            this.addRemoteOJProblemLoading = false;
          }
      );
    },
    changeContestProblemColor(id, color) {
      let data = {
        id: id,
        color: color,
      };
      api.admin_setContestProblemInfo(data).then((res) => {
        myMessage.success(this.$i18n.t('m.Update_Balloon_Color_Successfully'));
      });
    },
  },
  watch: {
    $route(newVal, oldVal) {
      this.init();
    },
  },
};
</script>

<style scoped>
.filter-row span button {
  margin-top: 5px;
  margin-bottom: 5px;
}

.filter-row span div {
  margin-top: 8px;
}

@media screen and (max-width: 768px) {
  .filter-row span {
    margin-right: 5px;
  }

  .filter-row span div {
    width: 80%;
  }
}

@media screen and (min-width: 768px) {
  .filter-row span {
    margin-right: 20px;
  }
}
</style>
