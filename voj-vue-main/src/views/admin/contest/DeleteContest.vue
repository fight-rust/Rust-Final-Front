<template>
  <div class="view">
    <el-card>
      <div slot="header">
        <span class="panel-title home-title">
          删除比赛
        </span>
      </div>
      <el-form label-position="top">
        <el-row :gutter="20">
          <el-col :span="24">
            <el-form-item :label="$t('m.Delete_Contest_Title')" required>
              <el-input
                  v-model="deleteid"
                  :placeholder="$t('m.Delete_Contest_placeholder')"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <!--账号限制规则-->
        <el-row :gutter="20">
          <template v-if="contest.openAccountLimit">
            <el-form :model="formRule">
              <el-col :md="6" :xs="24">
                <el-form-item :label="$t('m.Prefix')" prop="prefix">
                  <el-input
                      v-model="formRule.prefix"
                      placeholder="Prefix"
                  ></el-input>
                </el-form-item>
              </el-col>
              <el-col :md="6" :xs="24">
                <el-form-item :label="$t('m.Suffix')" prop="suffix">
                  <el-input
                      v-model="formRule.suffix"
                      placeholder="Suffix"
                  ></el-input>
                </el-form-item>
              </el-col>
              <el-col :md="6" :xs="24">
                <el-form-item :label="$t('m.Start_Number')" prop="number_from">
                  <el-input-number
                      v-model="formRule.number_from"
                      style="width: 100%"
                  ></el-input-number>
                </el-form-item>
              </el-col>
              <el-col :md="6" :xs="24">
                <el-form-item :label="$t('m.End_Number')" prop="number_to">
                  <el-input-number
                      v-model="formRule.number_to"
                      style="width: 100%"
                  ></el-input-number>
                </el-form-item>
              </el-col>

              <div
                  v-if="formRule.number_from <= formRule.number_to"
                  class="userPreview"
              >
                {{ $t('m.The_allowed_account_will_be') }}
                {{ formRule.prefix + formRule.number_from + formRule.suffix }},
                <span v-if="formRule.number_from + 1 < formRule.number_to">
                  {{
                    formRule.prefix +
                    (formRule.number_from + 1) +
                    formRule.suffix +
                    '...'
                  }}
                </span>
                <span v-if="formRule.number_from + 1 <= formRule.number_to">
                  {{ formRule.prefix + formRule.number_to + formRule.suffix }}
                </span>
              </div>

              <el-col :md="24" :xs="24">
                <el-form-item :label="$t('m.Extra_Account')" prop="prefix">
                  <el-input
                      v-model="formRule.extra_account"
                      :placeholder="$t('m.Extra_Account_Tips')"
                      :rows="8"
                      type="textarea"
                  >
                  </el-input>
                </el-form-item>
              </el-col>
            </el-form>
          </template>
        </el-row>
      </el-form>
      <el-button type="primary" @click.native="deleteContest">{{
          $t('m.DeleteContest')
        }}
      </el-button>
    </el-card>
  </div>
</template>

<script>
import api from '@/common/api';
import time from '@/common/time';
import moment from 'moment';
import {mapGetters} from 'vuex';
import myMessage from '@/common/message';

const Editor = () => import('@/components/admin/Editor.vue');
export default {
  name: 'CreateContest',
  components: {
    Editor,
  },
  data() {
    return {
      title: 'Create Contest',
      disableRuleType: false,
      durationText: '', // 比赛时长文本表示
      seal_rank_time: 2, // 当开启封榜模式，即实时榜单关闭时，可选择前半小时，前一小时，全程封榜,默认全程封榜
      contestid:[],
      deleteid:'',
      contest: {
        title: '',
        startTime: '',
        endTime: '',
        problem:'',
      },
      formRule: {
        prefix: '',
        suffix: '',
        number_from: 0,
        number_to: 10,
        extra_account: '',
      },
      starUserInput: '',
      inputVisible: false,
    };
  },
  created() {
    api.admin_getContestList().then(
        (res) => {
          for(let i=0;i<res.data.cid.length;i++){
            this.contestid[i]=res.data.cid[i];
          }
        }
    );
  },
  mounted() {
    if (this.$route.name === 'admin-edit-contest') {
      this.title = this.$i18n.t('m.Edit_Contest');
      this.disableRuleType = true;
      this.getContestByCid();
    } else {
      this.title = this.$i18n.t('m.Create_Contest');
      this.disableRuleType = false;
    }
  },
  watch: {
    $route() {
      if (this.$route.name === 'admin-edit-contest') {
        this.title = this.$i18n.t('m.Edit_Contest');
        this.disableRuleType = true;
        this.getContestByCid();
      } else {
        this.title = this.$i18n.t('m.Create_Contest');
        this.disableRuleType = false;
        this.contest = {};
      }
    },
  },
  computed: {
    ...mapGetters(['userInfo']),
  },
  methods: {
    getContestByCid() {
      api
          .admin_getContest(this.$route.params.contestId)
          .then((res) => {
            let data = res.data.data;
            this.contest = data;
            this.changeDuration();
            // 封榜时间转换
            let halfHour = moment(this.contest.endTime)
                .subtract(1800, 'seconds')
                .toString();
            let oneHour = moment(this.contest.endTime)
                .subtract(3600, 'seconds')
                .toString();
            let allHour = moment(this.contest.startTime).toString();
            let sealRankTime = moment(this.contest.sealRankTime).toString();
            switch (sealRankTime) {
              case halfHour:
                this.seal_rank_time = 0;
                break;
              case oneHour:
                this.seal_rank_time = 1;
                break;
              case allHour:
                this.seal_rank_time = 2;
                break;
            }
            if (this.contest.accountLimitRule) {
              this.formRule = this.changeStrToAccountRule(
                  this.contest.accountLimitRule
              );
            }
          })
          .catch(() => {
          });
    },

    deleteContest() {
      let flag=1;
      let temp=this.deleteid.split(';');
      let temp2=[];

      for(let i=0;i<temp.length;i++){
        let num=parseInt(temp[i]);
        if(isNaN(num)){
          myMessage.error("题目输入格式有误或题目id非数字！")
          flag=0;
          break;
        }
        else if(this.contestid.find(item=>item===num)===undefined){
          myMessage.error("id对应的比赛不存在！")
          flag=0;
          break;
        }
        temp2.push(num);
      }
      let temp3={deleteid:temp2};

      if(flag===1){
        api.admin_deleteContest(temp3)
            .then((res) => {
              myMessage.success('success');
              this.$router.push({
                name: 'admin-contest-list',
                query: {refresh: 'true'},
              });
            })
            .catch(() => {
            });
      }
    },

    changeDuration() {
      let start = this.contest.startTime;
      let end = this.contest.endTime;
      let durationMS = time.durationMs(start, end);
      if (durationMS < 0) {
        this.durationText = this.$i18n.t('m.Contest_Time_Check');
        this.contest.duration = 0;
        return;
      }
      if (start != '' && end != '') {
        this.durationText = time.formatSpecificDuration(start, end);
        this.contest.duration = durationMS;
      }
    },

    changeAccountRuleToStr(formRule) {
      let result =
          '<prefix>' +
          formRule.prefix +
          '</prefix><suffix>' +
          formRule.suffix +
          '</suffix><start>' +
          formRule.number_from +
          '</start><end>' +
          formRule.number_to +
          '</end><extra>' +
          formRule.extra_account +
          '</extra>';
      return result;
    },

    changeStrToAccountRule(value) {
      let reg =
          '<prefix>([\\s\\S]*?)</prefix><suffix>([\\s\\S]*?)</suffix><start>([\\s\\S]*?)</start><end>([\\s\\S]*?)</end><extra>([\\s\\S]*?)</extra>';
      let re = RegExp(reg, 'g');
      let tmp = re.exec(value);
      return {
        prefix: tmp[1],
        suffix: tmp[2],
        number_from: tmp[3],
        number_to: tmp[4],
        extra_account: tmp[5],
      };
    },

    addStarUser() {
      this.starUserInput = this.starUserInput.replace(/(^\s*)|(\s*$)/g, '');
      if (this.starUserInput) {
        for (var i = 0; i < this.contest.starAccount.length; i++) {
          if (this.contest.starAccount[i] == this.starUserInput) {
            myMessage.warning(this.$i18n.t('m.Add_Star_User_Error'));
            this.starUserInput = '';
            return;
          }
        }
        this.contest.starAccount.push(this.starUserInput);
        this.inputVisible = false;
        this.starUserInput = '';
      }
    },

    // 根据UserName 从打星用户列表中移除
    removeStarUser(username) {
      this.contest.starAccount.splice(
          this.contest.starAccount.map((item) => item.name).indexOf(username),
          1
      );
    },

    setSealRankTimeDefaultValue() {
      if (this.contest.sealRank == true) {
        if (this.contest.type == 0) {
          // ACM比赛开启封榜 默认为一小时,如果比赛时长小于一小时，则默认为全程
          if (this.contest.duration < 3600) {
            this.seal_rank_time = 2;
          } else {
            this.seal_rank_time = 1;
          }
        } else {
          // OI比赛开启封榜 默认全程
          this.seal_rank_time = 2;
        }
      }
    },
  },
};
</script>

<style scoped>
.userPreview {
  padding-left: 10px;
  padding-top: 20px;
  padding-bottom: 20px;
  color: red;
  font-size: 16px;
  margin-bottom: 10px;
}

.input-new-star-user {
  width: 200px;
}
</style>
