<template>
  <div class="problem">
    <el-card>
      <div slot="header">
        <span class="panel-title home-title">{{ title }}增加题目</span>
      </div>
      <el-form
          ref="form"
          :model="problem"
          :rules="rules"
          label-position="top"
          label-width="70px"
      >
        <el-row :gutter="20">
          <el-col :span="24">
            <el-form-item
                :label="$t('m.Problem_Display_ID')"
                prop="problemId"
                required
            >
              <el-input
                  v-model="problem.problemId"
                  :disabled="problem.isRemote"
                  :placeholder="$t('m.Problem_Display_ID')"
              >
              </el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="24">
            <el-form-item :label="$t('m.Title')" prop="title" required>
              <el-input
                  v-model="problem.title"
                  :placeholder="$t('m.Title')"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="24">
            <el-form-item :label="$t('m.pExample')" prop="title" required>
              <el-input
                  v-model="problem.title"
                  :placeholder="$t('m.pExample')"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-button size="small" type="primary" @click.native="submit()">{{
            $t('m.Save')
          }}
        </el-button>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import utils from '@/common/utils';
import {mapGetters} from 'vuex';
import api from '@/common/api';
import myMessage from '@/common/message';
import {PROBLEM_LEVEL, JUDGE_CASE_MODE} from '@/common/constants';

const Editor = () => import('@/components/admin/Editor.vue');
const Accordion = () => import('@/components/admin/Accordion.vue');
const AddExtraFile = () => import('@/components/admin/AddExtraFile.vue');
const CodeMirror = () => import('@/components/admin/CodeMirror.vue');
export default {
  name: 'Problem',
  components: {
    Accordion,
    AddExtraFile,
    CodeMirror,
    Editor,
  },
  data() {
    return {
      rules: {
        title: {
          required: true,
          message: 'Title is required',
          trigger: 'blur',
        },
        input_description: {
          required: true,
          message: 'Input Description is required',
          trigger: 'blur',
        },
        output_description: {
          required: true,
          message: 'Output Description is required',
          trigger: 'blur',
        },
      },
      backPath: '',
      loadingCompile: false,
      mode: '', // 该题目是编辑或者创建
      contest: {},
      codeTemplate: {},
      pid: null, // 题目id，如果为创建模式则为null
      contestID: null, // 比赛id
      contestProblem: {
        displayId: null,
        displayTitle: null,
        cid: null,
        pid: null,
      }, // 比赛题目的相关属性
      problem: {
        id: null,
        title: '',
        problemId: '',
        description: '',
        input: '',
        output: '',
        timeLimit: 1000,
        memoryLimit: 256,
        stackLimit: 128,
        difficulty: 1,
        auth: 3,
        codeShare: true,
        examples: [], // 题面上的样例输入输出
        spjLanguage: '',
        spjCode: '',
        spjCompileOk: false,
        uploadTestcaseDir: '',
        testCaseScore: [],
        isRemote: false,
        isRemoveEndBlank: true,
        openCaseResult: true,
        isUploadCase: true,
        type: 0,
        hint: '',
        source: '',
        cid: null,
        judgeMode: 'default',
        judgeCaseMode: 'default',
        userExtraFile: '',
        judgeExtraFile: '',
      },
      problemTags: [], //指定问题的标签列表
      problemLanguages: [], //指定问题的编程语言列表
      problemSamples: [], // 判题机使用的样例
      problemCodeTemplate: [],
      reProblem: {},
      testCaseUploaded: false,
      allLanguage: [], //所有编程语言
      allSpjLanguage: [], // 所有可以用特殊判题的语言
      allTags: [],
      allTagsTmp: [],
      inputVisible: false,
      tagInput: '',
      title: '',
      spjMode: '',
      disableRuleType: false,
      routeName: '',
      uploadTestcaseDir: '',
      uploadFileUrl: '',
      error: {
        tags: '',
        spj: '',
        languages: '',
        testCase: '',
      },
      PROBLEM_LEVEL: {},
      JUDGE_CASE_MODE: {},
      spjRecord: {
        spjCode: '',
        spjLanguage: '',
      },
      addUserExtraFile: false,
      addJudgeExtraFile: false,
      userExtraFile: null,
      judgeExtraFile: null,
    };
  },
  mounted() {
    this.PROBLEM_LEVEL = Object.assign({}, PROBLEM_LEVEL);
    this.JUDGE_CASE_MODE = Object.assign({}, JUDGE_CASE_MODE);
    this.routeName = this.$route.name;
    let contestID = this.$route.params.contestId;
    this.uploadFileUrl = '/api/file/upload-testcase-zip';
    if (
        this.routeName === 'admin-edit-problem' ||
        this.routeName === 'admin-edit-contest-problem'
    ) {
      this.mode = 'edit';
    } else {
      this.mode = 'add';
    }
    api
        .admin_getAllProblemTagList('ALL')
        .then((res) => {
          this.allTags = res.data.data;
          for (let tag of res.data.data) {
            this.allTagsTmp.push({value: tag.name, oj: tag.oj});
          }
        })
        .catch(() => {
        });
    api.getLanguages(this.$route.params.problemId, false).then((res) => {
      let allLanguage = res.data.data;
      this.allLanguage = allLanguage;
      for (let i = 0; i < allLanguage.length; i++) {
        if (allLanguage[i].isSpj == true) {
          this.allSpjLanguage.push(allLanguage[i]);
        }
      }
      this.problem = this.reProblem = {
        id: null,
        problemId: '',
        title: '',
        description: '',
        input: '',
        output: '',
        timeLimit: 1000,
        memoryLimit: 256,
        stackLimit: 128,
        difficulty: 1,
        auth: 3,
        codeShare: false,
        examples: [],
        spjLanguage: '',
        spjCode: '',
        spjCompileOk: false,
        isRemoveEndBlank: true,
        openCaseResult: true,
        isUploadCase: true,
        uploadTestcaseDir: '',
        testCaseScore: [],
        contestProblem: {},
        type: 1,
        hint: '',
        source: '',
        cid: null,
        judgeMode: 'default',
        judgeCaseMode: 'default',
        userExtraFile: null,
        judgeExtraFile: null,
      };

      this.contestID = contestID;
      if (contestID) {
        this.problem.cid = this.reProblem.cid = contestID;
        this.problem.auth = this.reProblem.auth = 3;
        this.disableRuleType = true;
        api.admin_getContest(contestID).then((res) => {
          this.problem.type = this.reProblem.type = res.data.data.type;
          this.contest = res.data.data;
        });
      }
      this.problem.spjLanguage = 'C';
      this.init();
    });
  },
  watch: {
    $route() {
      this.routeName = this.$route.name;
      if (
          this.routeName === 'admin-edit-problem' ||
          this.routeName === 'admin-edit-contest-problem'
      ) {
        this.mode = 'edit';
      } else {
        this.mode = 'add';
      }
      this.$refs.form.resetFields();
      this.problem = this.reProblem;
      this.problemTags = []; //指定问题的标签列表
      this.problemLanguages = []; //指定问题的编程语言列表
      this.problemSamples = [];
      this.problemCodeTemplate = [];
      this.codeTemplate = [];
      this.init();
    },

    problemLanguages(newVal) {
      let data = {};
      // use deep copy to avoid infinite loop
      let languages = JSON.parse(JSON.stringify(newVal)).sort();
      for (let item of languages) {
        if (this.codeTemplate[item] === undefined) {
          let langConfig = this.allLanguage.find((lang) => {
            return lang.name === item;
          });
          let codeTemp;
          let problemCodeTemplate = this.problemCodeTemplate;
          if (problemCodeTemplate) {
            codeTemp = problemCodeTemplate.find((temp) => {
              return temp.lid == langConfig.id;
            });
          }
          if (codeTemp == undefined) {
            data[item] = {
              id: null,
              status: false,
              code: langConfig.codeTemplate,
              mode: langConfig.contentType,
            };
          } else {
            data[item] = {
              id: codeTemp.id,
              status: true,
              code: codeTemp.code,
              mode: langConfig.contentType,
            };
          }
        } else {
          data[item] = this.codeTemplate[item];
        }
      }
      this.codeTemplate = data;
    },

    'problem.spjLanguage'(newVal) {
      if (this.allSpjLanguage.length) {
        this.spjMode = this.allSpjLanguage.find((item) => {
          return item.name == this.problem.spjLanguage && item.isSpj == true;
        })['contentType'];
      }
    },
  },
  methods: {
    init() {
      if (this.mode === 'edit') {
        this.pid = this.$route.params.problemId;
        this.backPath = this.$route.query.back;
        this.title = this.$i18n.t('m.Edit_Problem');
        let funcName = {
          'admin-edit-problem': 'admin_getProblem',
          'admin-edit-contest-problem': 'admin_getContestProblem',
        }[this.routeName];
        api[funcName](this.pid).then((problemRes) => {
          let data = problemRes.data.data;
          data.spjCompileOk = false;
          data.uploadTestcaseDir = '';
          data.testCaseScore = [];
          if (!data.spjCode) {
            data.spjCode = '';
          }
          data.spjLanguage = data.spjLanguage || 'C';
          this.spjRecord.spjLanguage = data.spjLanguage;
          this.spjRecord.spjCode = data.spjCode;
          this.problem = data;
          this.problem['examples'] = utils.stringToExamples(data.examples);
          this.problem['examples'][0]['isOpen'] = true;
          this.testCaseUploaded = true;
          if (this.problem.userExtraFile) {
            this.addUserExtraFile = true;
            this.userExtraFile = JSON.parse(this.problem.userExtraFile);
          }
          if (this.problem.judgeExtraFile) {
            this.addJudgeExtraFile = true;
            this.judgeExtraFile = JSON.parse(this.problem.judgeExtraFile);
          }
          api
              .admin_getProblemCases(this.pid, this.problem.isUploadCase)
              .then((res) => {
                if (this.problem.isUploadCase) {
                  this.problem.testCaseScore = res.data.data;
                } else {
                  this.problemSamples = res.data.data;
                  this.problemSamples[0]['isOpen'] = true;
                }
              });
        });
        if (funcName === 'admin_getContestProblem') {
          api
              .admin_getContestProblemInfo(this.pid, this.contestID)
              .then((res) => {
                this.contestProblem = res.data.data;
              });
        }
        this.getProblemCodeTemplateAndLanguage();

        api.admin_getProblemTags(this.pid).then((res) => {
          this.problemTags = res.data.data;
        });
      } else {
        this.addExample();
        this.testCaseUploaded = false;
        this.title = this.$i18n.t('m.Create_Problem');
        for (let item of this.allLanguage) {
          this.problemLanguages.push(item.name);
        }
      }
    },

    async getProblemCodeTemplateAndLanguage() {
      const that = this;
      await api.getProblemCodeTemplate(that.pid).then((res) => {
        that.problemCodeTemplate = res.data.data;
      });
      api.getProblemLanguages(that.pid).then((res) => {
        let Languages = res.data.data;
        for (let i = 0; i < Languages.length; i++) {
          that.problemLanguages.push(Languages[i].name);
        }
      });
    },

    switchMode(mode) {
      let modeName = 'General_Judge';
      let modeTips = 'General_Judge_Mode_Tips';
      if (mode == 'spj') {
        modeName = 'Special_Judge';
        modeTips = 'Special_Judge_Mode_Tips';
      } else if (mode == 'interactive') {
        modeName = 'Interactive_Judge';
        modeTips = 'Interactive_Judge_Mode_Tips';
      }
      const h = this.$createElement;
      this.$msgbox({
        title: this.$i18n.t('m.' + modeName),
        message: h('div', null, [
          h(
              'p',
              {style: 'text-align: center;font-weight:bolder;color:red'},
              this.$i18n.t('m.Change_Judge_Mode')
          ),
          h('br', null, null),
          h(
              'p',
              {style: 'font-weight:bolder'},
              this.$i18n.t('m.' + modeTips)
          ),
        ]),
      });
    },
    querySearch(queryString, cb) {
      var ojName = 'LOCAL';
      if (this.problem.isRemote) {
        ojName = this.problem.problemId.split('-')[0];
      }
      var restaurants = this.allTagsTmp.filter((item) => item.oj == ojName);
      var results = queryString
          ? restaurants.filter(
              (item) =>
                  item.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
          )
          : restaurants;
      cb(results);
    },
    changeContent(newVal) {
      this.announcement.content = newVal;
    },
    getLevelName(difficulty) {
      return utils.getLevelName(difficulty);
    },

    selectTag(item) {
      for (var i = 0; i < this.problemTags.length; i++) {
        if (this.problemTags[i].name == item.value) {
          myMessage.warning(this.$i18n.t('m.Add_Tag_Error'));
          this.tagInput = '';
          return;
        }
      }
      this.tagInput = item.value;
    },
    addTag(item) {
      let newTag = {
        name: this.tagInput,
      };
      if (this.tagInput) {
        for (var i = 0; i < this.problemTags.length; i++) {
          if (this.problemTags[i].name == this.tagInput) {
            myMessage.warning(this.$i18n.t('m.Add_Tag_Error'));
            this.tagInput = '';
            return;
          }
        }
        this.problemTags.push(newTag);
        this.inputVisible = false;
        this.tagInput = '';
      }
    },

    // 根据tag name从题目的tags列表中移除
    closeTag(tag) {
      this.problemTags.splice(
          this.problemTags.map((item) => item.name).indexOf(tag),
          1
      );
    },

    deleteFile(type, name) {
      if (type == 'user') {
        this.$delete(this.userExtraFile, name);
      } else {
        this.$delete(this.judgeExtraFile, name);
      }
    },

    upsertFile(type, name, oldname, content) {
      if (type == 'user') {
        if (oldname && oldname != name) {
          this.$delete(this.userExtraFile, oldname);
        }
        if (!this.userExtraFile) {
          this.userExtraFile = {};
        }
        this.userExtraFile[name] = content;
      } else {
        if (oldname && oldname != name) {
          this.$delete(this.judgeExtraFile, name);
        }
        if (!this.judgeExtraFile) {
          this.judgeExtraFile = {};
        }
        this.judgeExtraFile[name] = content;
      }
    },

    problemTypeChange(type) {
      if (type == 1) {
        let length = this.problemSamples.length;
        let aver = parseInt(100 / length);
        let add_1_num = 100 - aver * length;
        for (let i = 0; i < length; i++) {
          if (i >= length - add_1_num) {
            this.problemSamples[i].score = aver + 1;
          } else {
            this.problemSamples[i].score = aver;
          }
        }
      }
    },

    // 添加题目样例
    addExample() {
      this.problem.examples.push({input: '', output: '', isOpen: true});
    },
    changeExampleVisible(index, isOpen) {
      this.problem.examples[index]['isOpen'] = isOpen;
    },
    // 添加判题机的测试样例
    addSample() {
      if (this.mode === 'edit') {
        this.problemSamples.push({
          input: '',
          output: '',
          score: this.problem.type == 0 ? null : 0,
          pid: this.pid,
          isOpen: true,
        });
      } else {
        this.problemSamples.push({
          input: '',
          output: '',
          score: this.problem.type == 0 ? null : 0,
          pid: this.pid,
          isOpen: true,
        });
      }
    },
    //根据下标删除特定的题目样例
    deleteExample(index) {
      this.problem.examples.splice(index, 1);
    },
    //根据下标删除特定的判题机测试样例
    deleteSample(index) {
      this.problemSamples.splice(index, 1);
    },
    changeSampleVisible(index, isOpen) {
      this.problemSamples[index]['isOpen'] = isOpen;
    },
    uploadSucceeded(response) {
      if (response.status != 200) {
        myMessage.error(response.msg);
        this.testCaseUploaded = false;
        return;
      }
      myMessage.success(this.$i18n.t('m.Upload_Testcase_Successfully'));
      let fileList = response.data.fileList;
      let averScore = parseInt(100 / fileList.length);
      let add_1_num = 100 - averScore * fileList.length;
      for (let i = 0; i < fileList.length; i++) {
        if (averScore) {
          if (i >= fileList.length - add_1_num) {
            fileList[i].score = averScore + 1;
          } else {
            fileList[i].score = averScore;
          }
        }
        if (!fileList[i].output) {
          fileList[i].output = '-';
        }
        fileList[i].pid = this.problem.id;
      }
      this.problem.testCaseScore = fileList;
      this.testCaseUploaded = true;
      this.problem.uploadTestcaseDir = response.data.fileListDir;
    },
    uploadFailed() {
      myMessage.error(this.$i18n.t('m.Upload_Testcase_Failed'));
    },

    compileSPJ() {
      let data = {
        pid: this.problem.id,
        code: this.problem.spjCode,
        language: this.problem.spjLanguage,
        extraFiles: this.judgeExtraFile,
      };
      this.loadingCompile = true;
      let apiMethodName = 'compileSPJ';
      if (this.problem.judgeMode == 'interactive') {
        apiMethodName = 'compileInteractive';
      }
      api[apiMethodName](data).then(
          (res) => {
            this.loadingCompile = false;
            this.problem.spjCompileOk = true;
            this.error.spj = '';
            myMessage.success(res.data.msg);
          },
          (err) => {
            this.loadingCompile = false;
            this.problem.spjCompileOk = false;
            const h = this.$createElement;
            this.$msgbox({
              title: 'Compile Error',
              type: 'error',
              message: h('pre', err.data.msg),
              showCancelButton: false,
              closeOnClickModal: false,
              customClass: 'dialog-compile-error',
            });
          }
      );
    },

    submit() {
      if (!this.problem.problemId) {
        myMessage.error(
            this.$i18n.t('m.Problem_Display_ID') +
            ' ' +
            this.$i18n.t('m.is_required')
        );
        return;
      }

      if (this.contestID) {
        if (!this.contestProblem.displayId) {
          myMessage.error(
              this.$i18n.t('m.Contest_Display_ID') +
              ' ' +
              this.$i18n.t('m.is_required')
          );
          return;
        }
        // if(!/^[0-9]*$/.test(this.contestProblem.displayId)){
        //   myMessage.error(
        //       this.$i18n.t('m.The_Problem_Display_ID_in_the_Contest_format_error')
        //   );
        //   return;
        // }
        if (!this.contestProblem.displayTitle) {
          myMessage.error(
              this.$i18n.t('m.Contest_Display_Title') +
              ' ' +
              this.$i18n.t('m.is_required')
          );
          return;
        }
      }

      // // 不强制校验题目样例不能为空
      // if (!this.problem.examples.length && !this.problem.isRemote) {
      //   myMessage.error(
      //     this.$i18n.t('m.Problem_Examples') +
      //       ' ' +
      //       this.$i18n.t('m.is_required')
      //   );
      //   return;
      // }

      if (!this.problem.isRemote) {
        // 选择手动输入
        if (!this.problem.isUploadCase) {
          if (!this.problemSamples.length) {
            myMessage.error(
                this.$i18n.t('m.Judge_Samples') +
                ' ' +
                this.$i18n.t('m.is_required')
            );
            return;
          }

          for (let sample of this.problemSamples) {
            if (!sample.input && !sample.output) {
              myMessage.error(
                  this.$i18n.t('m.Sample_Input') +
                  ' or ' +
                  this.$i18n.t('m.Sample_Output') +
                  ' ' +
                  this.$i18n.t('m.is_required')
              );
              return;
            }
          }

          // 同时是oi题目，则对应的每个测试样例的io得分不能为空或小于0
          if (this.problem.type == 1) {
            for (let i = 0; i < this.problemSamples.length; i++) {
              if (this.problemSamples[i].score == '') {
                myMessage.error(
                    this.$i18n.t('m.Problem_Sample') +
                    (i + 1) +
                    ' ' +
                    this.$i18n.t('m.Score_must_be_an_integer')
                );
                return;
              }
              try {
                if (parseInt(this.problemSamples[i].score) < 0) {
                  myMessage.error(
                      this.$i18n.t('m.Problem_Sample') +
                      (i + 1) +
                      ' ' +
                      this.$i18n.t('m.Score_must_be_greater_than_or_equal_to_0')
                  );
                  return;
                }
              } catch (e) {
                myMessage.error(this.$i18n.t('m.Score_must_be_an_integer'));
                return;
              }
            }
          }
        } else {
          // 选择上传文件

          // 两种情况：create模式是需要校验是否上传成功了，edit模式获取题目数据已经默认为true了，若是edit又重新上传数据，需要校验
          if (!this.testCaseUploaded) {
            this.error.testCase =
                this.$i18n.t('m.Judge_Samples') +
                ' ' +
                this.$i18n.t('m.is_required');
            myMessage.error(this.error.testCase);
            return;
          }

          // 如果是oi题目，需要检查上传的数据的得分
          if (this.problem.type == 1) {
            for (let i = 0; i < this.problemSamples.length; i++) {
              if (this.problemSamples[i].score == '') {
                myMessage.error(
                    this.$i18n.t('m.Problem_Sample') +
                    (i + 1) +
                    ' ' +
                    this.$i18n.t('m.Score_must_be_an_integer')
                );
                return;
              }
              try {
                if (parseInt(this.problemSamples[i].score) < 0) {
                  myMessage.error(
                      this.$i18n.t('m.Problem_Sample') +
                      (i + 1) +
                      ' ' +
                      this.$i18n.t('m.Score_must_be_greater_than_or_equal_to_0')
                  );
                  return;
                }
              } catch (e) {
                myMessage.error(this.$i18n.t('m.Score_must_be_an_integer'));
                return;
              }
            }
          }
        }
      }
      if (!this.problemTags.length) {
        this.error.tags =
            this.$i18n.t('m.Tags') + ' ' + this.$i18n.t('m.is_required');
        myMessage.error(this.error.tags);
        return;
      }
      let isChangeModeCode =
          this.spjRecord.spjLanguage != this.problem.spjLanguage ||
          this.spjRecord.spjCode != this.problem.spjCode;
      if (!this.problem.isRemote) {
        if (this.problem.judgeMode != 'default') {
          if (!this.problem.spjCode) {
            this.error.spj =
                this.$i18n.t('m.Spj_Or_Interactive_Code') +
                ' ' +
                this.$i18n.t('m.is_required');
            myMessage.error(this.error.spj);
          } else if (!this.problem.spjCompileOk && isChangeModeCode) {
            this.error.spj = this.$i18n.t(
                'm.Spj_Or_Interactive_Code_not_Compile_Success'
            );
          }
          if (this.error.spj) {
            myMessage.error(this.error.spj);
            return;
          }
        }
      }

      if (!this.problemLanguages.length) {
        this.error.languages =
            this.$i18n.t('m.Language') + ' ' + this.$i18n.t('m.is_required');
        myMessage.error(this.error.languages);
        return;
      }

      let funcName = {
        'admin-create-problem': 'admin_createProblem',
        'admin-edit-problem': 'admin_editProblem',
        'admin-create-contest-problem': 'admin_createContestProblem',
        'admin-edit-contest-problem': 'admin_editContestProblem',
      }[this.routeName];
      // edit contest problem 时, contest_id会被后来的请求覆盖掉
      if (funcName === 'editContestProblem') {
        this.problem.cid = this.contest.id;
      }
      if (
          funcName === 'admin_createProblem' ||
          funcName === 'admin_createContestProblem'
      ) {
        this.problem.author = this.userInfo.username;
      }

      var ojName = 'LOCAL';
      if (this.problem.isRemote) {
        ojName = this.problem.problemId.split('-')[0];
      }

      let problemTagList = Object.assign([], this.problemTags);
      for (let i = 0; i < problemTagList.length; i++) {
        //避免后台插入违反唯一性
        for (let tag2 of this.allTags) {
          if (problemTagList[i].name == tag2.name && tag2.oj == ojName) {
            problemTagList[i] = tag2;
            break;
          }
        }
      }
      this.problemCodeTemplate = [];
      let problemLanguageList = Object.assign([], this.problemLanguages); // 深克隆 防止影响
      for (let i = 0; i < problemLanguageList.length; i++) {
        problemLanguageList[i] = {name: problemLanguageList[i]};
        for (let lang of this.allLanguage) {
          if (problemLanguageList[i].name == lang.name) {
            problemLanguageList[i] = lang;
            if (this.codeTemplate[lang.name].status) {
              this.problemCodeTemplate.push({
                id: this.codeTemplate[lang.name].id,
                pid: this.pid,
                code: this.codeTemplate[lang.name].code,
                lid: lang.id,
                status: this.codeTemplate[lang.name].status,
              });
            }
            break;
          }
        }
      }
      let problemDto = {}; // 上传给后台的数据
      if (!this.problem.isRemote) {
        if (this.problem.judgeMode != 'default') {
          if (isChangeModeCode) {
            problemDto['changeModeCode'] = true;
          }
        } else {
          // 原本是spj或交互，但现在关闭了
          if (!this.spjRecord.spjCode) {
            problemDto['changeModeCode'] = true;
            this.problem.spjCode = null;
            this.problem.spjLanguage = null;
          }
        }

        if (this.userExtraFile && Object.keys(this.userExtraFile).length != 0) {
          this.problem.userExtraFile = JSON.stringify(this.userExtraFile);
        } else {
          this.problem.userExtraFile = null;
        }

        if (
            this.judgeExtraFile &&
            Object.keys(this.judgeExtraFile).length != 0
        ) {
          this.problem.judgeExtraFile = JSON.stringify(this.judgeExtraFile);
        } else {
          this.problem.judgeExtraFile = null;
        }
      }

      problemDto['problem'] = Object.assign({}, this.problem); // 深克隆
      problemDto.problem.examples = utils.examplesToString(
          this.problem.examples
      ); // 需要转换格式

      problemDto['codeTemplates'] = this.problemCodeTemplate;
      problemDto['tags'] = problemTagList;
      problemDto['languages'] = problemLanguageList;
      problemDto['isUploadTestCase'] = this.problem.isUploadCase;
      problemDto['uploadTestcaseDir'] = this.problem.uploadTestcaseDir;
      problemDto['judgeMode'] = this.problem.judgeMode;

      // 如果选择上传文件，则使用上传后的结果
      if (this.problem.isUploadCase) {
        problemDto['samples'] = this.problem.testCaseScore;
      } else {
        problemDto['samples'] = this.problemSamples;
      }
      api[funcName](problemDto)
          .then((res) => {
            if (
                this.routeName === 'admin-create-contest-problem' ||
                this.routeName === 'admin-edit-contest-problem'
            ) {
              if (res.data.data) {
                // 新增题目操作 需要使用返回来的pid
                this.contestProblem['pid'] = res.data.data.pid;
                this.contestProblem['cid'] = this.$route.params.contestId;
              }
              api.admin_setContestProblemInfo(this.contestProblem).then((res) => {
                myMessage.success('success');
                this.$router.push({
                  name: 'admin-contest-problem-list',
                  params: {contestId: this.$route.params.contestId},
                });
              });
            } else {
              myMessage.success('success');
              if (this.backPath) {
                this.$router.push({path: this.backPath});
              } else {
                this.$router.push({name: 'admin-problem-list'});
              }
            }
          })
          .catch(() => {
          });
    },
  },
  computed: {
    ...mapGetters(['userInfo']),
  },
};
</script>

<style scoped>
/deep/ .el-form-item__label {
  padding: 0 !important;
}

.el-form-item {
  margin-bottom: 10px !important;
}

.difficulty-select {
  width: 120px;
}

.input-new-tag {
  width: 120px;
}

.button-new-tag {
  height: 24px;
  line-height: 22px;
  padding-top: 0;
  padding-bottom: 0;
}

.accordion {
  margin-bottom: 10px;
}

.add-examples {
  width: 100%;
  background-color: #fff;
  border: 1px dashed #2d8cf0;
  outline: none;
  cursor: pointer;
  color: #2d8cf0;
  height: 35px;
  font-size: 14px;
}

.add-examples i {
  margin-right: 10px;
}

.add-examples:hover {
  border: 0px;
  background-color: #2d8cf0 !important;
  color: #fff;
}

.add-example-btn {
  margin-bottom: 10px;
}

.add-samples {
  width: 100%;
  background-color: #fff;
  border: 1px dashed #19be6b;
  outline: none;
  cursor: pointer;
  color: #19be6b;
  height: 35px;
  font-size: 14px;
}

.add-samples i {
  margin-right: 10px;
}

.add-samples:hover {
  border: 0px;
  background-color: #19be6b !important;
  color: #fff;
}

.add-sample-btn {
  margin-bottom: 10px;
}

.dialog-compile-error {
  width: auto;
  max-width: 80%;
  overflow-x: scroll;
}
</style>
