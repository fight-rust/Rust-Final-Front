<template>
  <div>
    <el-card>
      <div slot="header">
        <span class="panel-title home-title">{{ $t('m.Contest_List') }}</span>

      </div>
      <vxe-table
          ref="xTable"
          :data="contestList"
          :loading="loading"
          align="center"
          auto-resize
          stripe
      >
        <vxe-table-column field="cid" title="比赛ID" width="80"></vxe-table-column>
        <vxe-table-column
            :title="$t('m.Title')"
            field="ctitle"
            min-width="150"
        >
        </vxe-table-column>
      </vxe-table>
    </el-card>
    <el-dialog
        :title="$t('m.Download_Contest_AC_Submission')"
        :visible.sync="downloadDialogVisible"
        width="320px"
    >
      <el-switch
          v-model="excludeAdmin"
          :active-text="$t('m.Exclude_admin_submissions')"
      ></el-switch>
      <el-radio-group v-model="splitType" style="margin-top:10px">
        <el-radio label="user">{{ $t('m.SplitType_User') }}</el-radio>
        <el-radio label="problem">{{ $t('m.SplitType_Problem') }}</el-radio>
      </el-radio-group>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="downloadSubmissions">{{
            $t('m.OK')
          }}</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import api from '@/common/api';
import utils from '@/common/utils';
import {CONTEST_STATUS_REVERSE, CONTEST_TYPE_REVERSE,} from '@/common/constants';
import {mapGetters} from 'vuex';
import myMessage from '@/common/message';

export default {
  name: 'ContestList',
  data() {
    return {
      pageSize: 10,
      total: 0,
      contestList: [],
      keyword: '',
      loading: false,
      excludeAdmin: true,
      splitType: 'user',
      currentPage: 1,
      currentId: 1,
      downloadDialogVisible: false,
      CONTEST_TYPE_REVERSE: {},
    };
  },
  mounted() {
    this.CONTEST_TYPE_REVERSE = Object.assign({}, CONTEST_TYPE_REVERSE);
    this.CONTEST_STATUS_REVERSE = Object.assign({}, CONTEST_STATUS_REVERSE);
    this.getContestList(this.currentPage);
  },
  watch: {
    $route() {
      let refresh = this.$route.query.refresh == 'true' ? true : false;
      if (refresh) {
        this.getContestList(1);
      }
    },
  },
  computed: {
    ...mapGetters(['isSuperAdmin', 'userInfo']),
  },
  methods: {
    // 切换页码回调
    currentChange(page) {
      this.currentPage = page;
      this.getContestList(page);
    },
    getContestList(page) {
      this.loading = true;
      api.admin_getContestList(page, this.pageSize, this.keyword).then(
          (res) => {
            this.loading = false;
            for(let i=0;i<res.data.cid.length;i++){
              let temp={cid:res.data.cid[i],ctitle:res.data.ctitle[i]};
              this.contestList.push(temp);
            }
          },
          (res) => {
            this.loading = false;
          }
      );
    },
    openDownloadOptions(contestId) {
      this.downloadDialogVisible = true;
      this.currentId = contestId;
    },
    downloadSubmissions() {
      let url = `/api/file/download-contest-ac-submission?cid=${this.currentId}&excludeAdmin=${this.excludeAdmin}&splitType=${this.splitType}`;
      utils.downloadFile(url);
      this.downloadDialogVisible = false;
    },
    goEdit(contestId) {
      this.$router.push({name: 'admin-edit-contest', params: {contestId}});
    },
    goContestAnnouncement(contestId) {
      this.$router.push({
        name: 'admin-contest-announcement',
        params: {contestId},
      });
    },
    goContestProblemList(contestId) {
      this.$router.push({
        name: 'admin-contest-problem-list',
        params: {contestId},
      });
    },
    deleteContest(contestId) {
      this.$confirm(this.$i18n.t('m.Delete_Contest_Tips'), 'Tips', {
        confirmButtonText: this.$i18n.t('m.OK'),
        cancelButtonText: this.$i18n.t('m.Cancel'),
        type: 'warning',
      }).then(() => {
        api.admin_deleteContest(contestId).then((res) => {
          myMessage.success(this.$i18n.t('m.Delete_successfully'));
          this.currentChange(1);
        });
      });
    },
    changeContestVisible(contestId, visible, uid) {
      api.admin_changeContestVisible(contestId, visible, uid).then((res) => {
        myMessage.success(this.$i18n.t('m.Update_Successfully'));
      });
    },
    filterByKeyword() {
      this.currentChange(1);
    },
  },
};
</script>
<style scoped>
.filter-row {
  margin-top: 10px;
}

@media screen and (max-width: 768px) {
  .filter-row span {
    margin-right: 5px;
  }

  .filter-row span div {
    width: 80% !important;
  }
}

@media screen and (min-width: 768px) {
  .filter-row span {
    margin-right: 20px;
  }
}

.el-tag--dark {
  border-color: #fff;
}
</style>
