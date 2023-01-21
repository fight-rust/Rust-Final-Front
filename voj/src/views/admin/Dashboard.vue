<template>
  <div>
    <el-row :gutter="20">
      <el-col :lg="8" :md="10" :xs="24">
        <el-card class="admin-info">
          <div slot="header">
            <el-row :gutter="10">
              <el-col :span="10">
                <el-image :src="userInfo.avatar" class="avatar" fit="fill">
                  <div slot="error" class="image-slot">
                    <i class="el-icon-picture-outline"></i>
                  </div>
                </el-image>
              </el-col>
              <el-col :span="14">
                <span class="panel-title admin-info-name">{{
                    userInfo.username
                  }}</span>
                <p>
                  {{
                    isSuperAdmin == true
                        ? $t('m.Super_Admin')
                        : isProblemAdmin == true
                            ? $t('m.All_Problem_Admin')
                            : $t('m.Admin')
                  }}
                </p>
              </el-col>
            </el-row>
          </div>
          <div class="last-info">
            <p class="last-info-title home-title">{{ $t('m.Last_Login') }}</p>
            <el-form class="last-info-body" label-width="80px">
              <el-form-item label="Time:">
                <span>{{ session.gmtCreate | localtime }}</span>
              </el-form-item>
              <el-form-item label="IP:">
                <span>{{ session.ip }}</span>
              </el-form-item>
              <el-form-item label="OS:">
                <span>{{ os }}</span>
              </el-form-item>
              <el-form-item label="Browser:">
                <span>{{ browser }}</span>
              </el-form-item>
            </el-form>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import {mapGetters} from 'vuex';
import browserDetector from 'browser-detect';
import api from '@/common/api';

const InfoCard = () => import('@/components/admin/infoCard.vue');

export default {
  name: 'dashboard',
  components: {
    InfoCard,
  },
  data() {
    return {
      infoData: {
        userNum: 0,
        recentContestNum: 0,
        todayJudgeNum: 0,
      },
      generalInfo: {
        backupCores: 0,
        backupPercentCpuLoad: '0%',
        backupPercentMemoryLoad: '0%',
        backupService: [],
        nacos: {},
      },
      judgeInfo: [],
      session: {},
    };
  },
  mounted() {
    this.refreshJudgeServerList();
    this.refreshGeneralSystemInfo();
    // 每5秒刷新判题机服务和后台服务的情况
    this.intervalId = setInterval(() => {
      this.refreshJudgeServerList();
      this.refreshGeneralSystemInfo();
    }, 5000);
    api.admin_getDashboardInfo().then(
        (resp) => {
          this.infoData = resp.data.data;
        },
        () => {
        }
    );

    api.getSessions(this.userInfo.uid).then(
        (resp) => {
          this.session = resp.data.data;
        },
        () => {
        }
    );
  },
  methods: {
    refreshJudgeServerList() {
      api.getJudgeServer().then(
          (res) => {
            this.judgeInfo = res.data.data;
          },
          (err) => {
            clearInterval(this.intervalId);
          }
      );
    },
    refreshGeneralSystemInfo() {
      api.admin_getGeneralSystemInfo().then(
          (res) => {
            this.generalInfo = res.data.data;
            this.generalInfo.backupService[0][
                'backupCores'
                ] = this.generalInfo.backupCores;
            this.generalInfo.backupService[0][
                'backupPercentCpuLoad'
                ] = this.generalInfo.backupPercentCpuLoad;
            this.generalInfo.backupService[0][
                'backupPercentMemoryLoad'
                ] = this.generalInfo.backupPercentMemoryLoad;
          },
          () => {
            clearInterval(this.intervalId);
          }
      );
    },
  },
  computed: {
    ...mapGetters(['userInfo', 'isSuperAdmin', 'isProblemAdmin']),
    https() {
      return document.URL.slice(0, 5) === 'https';
    },
    browser() {
      let b = browserDetector(this.session.userAgent);
      if (b.name && b.version) {
        return b.name + ' ' + b.version;
      } else {
        return 'Unknown';
      }
    },
    os() {
      let b = browserDetector(this.session.userAgent);
      return b.os ? b.os : 'Unknown';
    },
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.intervalId);
    next();
  },
};
</script>

<style scoped>
.admin-info {
  margin-bottom: 20px;
}

.admin-info-name {
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 10px;
  color: #409eff;
}

.admin-info .avatar {
  max-width: 100%;
  max-width: 100px;
  max-height: 100px;
}

.admin-info .last-info-title {
  font-size: 16px;
}

.el-form-item {
  margin-bottom: 5px;
}

.info-container {
  display: flex;
  justify-content: flex-start;
  flex-wrap: wrap;
}

.info-container .info-item {
  flex: 1 0 auto;
  min-width: 200px;
  margin-bottom: 10px;
}

/deep/ .el-tag--dark {
  border-color: #fff;
}

/deep/ .el-card__header {
  padding-bottom: 0;
}

@media screen and (min-width: 1150px) {
  /deep/ .vxe-table--body-wrapper {
    overflow-x: hidden !important;
  }
}
</style>
