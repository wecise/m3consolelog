<template>
  <el-container>
    <el-header>
      <el-input
        placeholder="请输入内容"
        v-model="search.name"
        class="input-with-select"
        clearable
        size="large"
        @keyup.enter.native="onSearch"
      >
        <el-select
          v-model="search.type"
          slot="prepend"
          placeholder="选择日志类型"
        >
          <el-option label="rule" value="rule"></el-option>
          <el-option label="serverjs" value="serverjs"></el-option>
          <el-option label="trigger" value="trigger"></el-option>
        </el-select>
        <el-button
          slot="append"
          icon="el-icon-search"
          @click="onSearch"
        ></el-button>
      </el-input>
    </el-header>
    <el-main>
      <div class="editor-toolbar">
        <el-row style="width:100%;">
            <el-col :span="2">
                <el-tooltip content="重新加载" open-delay="800">
                    <el-button type="text" @click="onLoad" icon="el-icon-refresh"></el-button>
                </el-tooltip>    

                <el-tooltip content="删除" open-delay="800">
                    <el-button type="text" @click="onDelete" icon="el-icon-close"></el-button>
                </el-tooltip>

                <el-tooltip content="清空" open-delay="800">
                    <el-button type="text" @click="onTruncate" icon="el-icon-delete"></el-button>
                </el-tooltip>
            </el-col>
            
            <el-col :span="19" style="padding-left:10px;">
                <el-checkbox-group v-model="consolelog.level" class="mx-consolelog-level" v-if="global">
                    <el-checkbox :label="k" :key="k" v-for="(v,k) in global.register.consolelog.level">{{v[1]}}</el-checkbox>
                </el-checkbox-group>
            </el-col>
            <el-col :span="3" style="text-align:right;">
                {{ control.ifRefresh==1 ? '自动刷新' : '关闭刷新' }}
                <el-switch
                    v-model="control.ifRefresh"
                    active-color="#13ce66"
                    inactive-color="#dddddd"
                    active-value="1"
                    inactive-value="0">
                </el-switch>
            
                <el-tooltip content="选择主题" placement="bottom" open-delay="800">
                    <el-dropdown style="padding-left:10px;">
                      <span class="el-dropdown-link">
                        <i class="el-icon-s-tools el-icon--right"></i>
                      </span>
                      <el-dropdown-menu slot="dropdown">
                        <el-dropdown-item
                          v-for="group in editor.theme.list"
                          :key="group.name"
                        >
                          <el-dropdown @command="onCommand">
                            <span class="el-dropdown-link">
                              {{ group.name }}
                              <i class="el-icon-arrow-down el-icon--right"></i>
                            </span>
                            <el-dropdown-menu slot="dropdown">
                              <el-dropdown-item
                                v-for="item in group.items"
                                :key="item.name"
                                :command="item.name"
                                >{{ item.name }}</el-dropdown-item
                              >
                            </el-dropdown-menu>
                          </el-dropdown>
                        </el-dropdown-item>
                      </el-dropdown-menu>
                    </el-dropdown>
                </el-tooltip>
            </el-col>
        </el-row>
        <!--div class="button-group">
          <el-button-group>
            <el-date-picker
              v-model="search.time.value"
              type="datetime"
              placeholder="选择日期时间"
              align="right"
              :picker-options="search.time.pickerOptions"
            >
            </el-date-picker>
          </el-button-group>

        </div-->
        
      </div>
      <editor
        v-model="editor.content"
        @init="editorInit"
        lang="toml"
        :theme="editor.theme.value"
        width="100%"
        height="calc(100vh - 190px)"
      ></editor>
    </el-main>
    <el-footer>
      {{ editor.info.page }} {{ editor.info.time }}
      {{ editor.info.currentPage }} {{ editor.info.count }}
    </el-footer>
  </el-container>
</template>

<script>
import _ from "lodash";
export default {
  name: "MainView",
  props: {
    global: Object,
  },
  components: {
    editor: require("vue2-ace-editor"),
  },
  data(){
    return{
      search: {
        type: "serverjs",
        name: "",
        time: {
          value: "",
          pickerOptions: {
            shortcuts: [
              {
                text: "今天",
                onClick(picker) {
                  picker.$emit("pick", new Date());
                },
              },
              {
                text: "昨天",
                onClick(picker) {
                  const date = new Date();
                  date.setTime(date.getTime() - 3600 * 1000 * 24);
                  picker.$emit("pick", date);
                },
              },
              {
                text: "一周前",
                onClick(picker) {
                  const date = new Date();
                  date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
                  picker.$emit("pick", date);
                },
              },
            ],
          },
        }
      },
      consolelog: {
          level: [],
          maxtime: 0,
          mintime: 0,
          position: "",
          limit: 100
      },
      options:{
          mode: "toml",
          theme: "tomorrow",
          readOnly: true
      },
      control: {
          ifRefresh: false
      },
      dt: {
        rows: [],
        columns: [],
        info: "",
      },
      editor: {
        content: "",
        info: {
          page: null,
          time: null,
          currentPage: 1,
          count: 0,
        },
        theme: {
          value: "chrome",
          list: [
            {
              name: "亮色",
              items: [
                { name: "chrome" },
                { name: "clouds" },
                { name: "crimson_editor" },
                { name: "dawn" },
                { name: "dreamweaver" },
                { name: "eclipse" },
                { name: "github" },
                { name: "iplastic" },
                { name: "solarized_light" },
                { name: "textmate" },
                { name: "tomorrow" },
                { name: "xcode" },
                { name: "kuroir" },
                { name: "katzenmilch" },
                { name: "sqlserver" },
              ],
            },
            {
              name: "暗色",
              items: [
                { name: "ambiance" },
                { name: "chaos" },
                { name: "clouds_midnight" },
                { name: "dracula" },
                { name: "cobalt" },
                { name: "gruvbox" },
                { name: "gob" },
                { name: "idle_fingers" },
                { name: "kr_theme" },
                { name: "merbivore" },
                { name: "merbivore_soft" },
                { name: "mono_industrial" },
                { name: "monokai" },
                { name: "pastel_on_dark" },
                { name: "solarized_dark" },
                { name: "terminal" },
                { name: "tomorrow_night" },
                { name: "tomorrow_night_blue" },
                { name: "tomorrow_night_bright" },
                { name: "tomorrow_night_eighties" },
                { name: "twilight" },
                { name: "vibrant_ink" },
              ],
            },
          ],
        }
      }
    }
  },
  watch:{
    "search.name"(val) {
      if (!val) {
        this.editor.content = "";
      }
    },
    'dt.rows':{
        handler(val,oldVal){
            if (val !== oldVal && !_.isEmpty(val)) {
              let preFix = ["级别", "时间", "摘要"].join("  ");
              let csv = _.concat(
                preFix,
                _.map(val, (v) => {
                  return `[${this.global.register.consolelog.level[v.level][1]}]  ${v.msg}`;
                })
              ).join("\n");
              this.editor.content = csv;
            } else {
              this.editor.content = "";
            }
            
        },
        deep:true
    },
    'control.ifRefresh':{
        handler(val){
            if(val==1) {
                window.intervalListener = setInterval(()=>{
                    this.onLoad();
                },3000);
                this.$message({
                    type: "info",
                    message: "自动刷新开启"
                })
            } else {
                clearInterval(window.intervalListener);
                this.$message({
                    type: "info",
                    message: "自动刷新关闭"
                })
            }
        }
    },
  },
  created(){
      
      this.consolelog.level = _.keys(this.global.register.consolelog.level);
  
      this.onLoad();

  },
  methods:{
      editorInit() {
        require("brace/ext/language_tools"); //language extension prerequsite...
        require("brace/mode/html");
        require("brace/mode/toml");
        require("brace/mode/javascript"); //language
        require("brace/mode/less");
        require("brace/theme/chrome");
        require("brace/snippets/javascript"); //snippet
      },
      onSearch() {
        this.onLoad();
      },
      onCommand(val) {
        require(`brace/theme/${val}`);
        this.editor.theme.value = val;
      },
      onLoad(){

        let term = encodeURIComponent( JSON.stringify({ type: this.search.type, name: this.search.name, log: this.consolelog }));
        this.m3.callFS("/matrix/consoleLog/traceLog.js", term).then( (rtn)=>{
            this.dt.rows = rtn.message;
        })

      },
      onDelete(){
          let term = encodeURIComponent( JSON.stringify({ type: this.search.type, name: this.search.name }));
          this.m3.callFS("/matrix/consoleLog/deleteLog.js", term).then( ()=>{
              this.dt.rows = [];
          })
      },
      onTruncate(){
          let term = encodeURIComponent( JSON.stringify({ type: this.search.type }));
          this.m3.callFS("/matrix/consoleLog/truncateLog.js", term).then( ()=>{
              this.dt.rows = [];
          })
      },
      initTheme(){}
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.el-header {
  line-height: 60px;
}

.el-main {
  margin: 0 20px;
  padding: 0 20px;
  background: #f2f2f2;
}

.el-footer {
  line-height: 60px;
  margin: 0 20px;
  background: #f2f2f2;
  text-align: left;
}

.editor-toolbar {
  float: left;
  width: 100%;
  height: 50px;
  line-height: 50px;
  display: flex;
}

.editor-toolbar > .button-group {
  width: 90%;
  text-align: left;
}

.el-button-group + .el-button-group {
  padding-left: 10px;
}

.editor-toolbar > .el-dropdown {
  text-align: right;
  width: 9%;
  margin-right: 10px;
}

.el-input-group__prepend > .el-select {
  width: 120px;
}
</style>
