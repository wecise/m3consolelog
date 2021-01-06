<template>
  <el-container>
    <el-header>
      <el-input placeholder="请输入内容" v-model="search.name" class="input-with-select" clearable size="large">
          <el-select v-model="search.type" slot="prepend" placeholder="选择日志类型">
            <el-option label="rule" value="rule"></el-option>
            <el-option label="serverjs" value="serverjs"></el-option>
            <el-option label="trigger" value="trigger"></el-option>
          </el-select>
        <el-button slot="append" icon="el-icon-search" @click="onSearch"></el-button>
      </el-input>
    </el-header>
    <el-main>
      <div class="editor-toolbar">
        <div class="button-group">
          <el-button-group>
            <el-button type="default" icon="el-icon-d-arrow-left">首页</el-button>
            <el-button type="default" >尾页<i class="el-icon-d-arrow-right el-icon--right"></i></el-button>
          </el-button-group>
          
          <el-button-group>
            <el-button type="default" icon="el-icon-caret-top">上一页</el-button>
            <el-button type="default">下一页<i class="el-icon-caret-bottom el-icon--right"></i></el-button>
          </el-button-group>
          
          <el-button-group>
            <el-button type="default" icon="el-icon-arrow-up">上一条</el-button>
            <el-button type="default">下一条<i class="el-icon-arrow-down el-icon--right"></i></el-button>
          </el-button-group>
        </div>
        <el-dropdown>
          <span class="el-dropdown-link">
            <i class="el-icon-s-tools el-icon--right"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item v-for="group in editor.theme.list" :key="group.name">
                <el-dropdown @command="onCommand">
                    <span class="el-dropdown-link">
                      {{group.name}} <i class="el-icon-arrow-down el-icon--right"></i>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                      <el-dropdown-item v-for="item in group.items" :key="item.name" :command="item.name">{{item.name}}</el-dropdown-item>
                    </el-dropdown-menu>
                  </el-dropdown>
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        
      </div>
      <editor v-model="editor.content" @init="editorInit" lang="toml" :theme="editor.theme.value" width="100%" height="calc(100vh - 210px)"></editor>
    </el-main>
    <el-footer>
        {{editor.info.page}} {{editor.info.time}} {{editor.info.currentPage}} {{editor.info.count}}
    </el-footer>
  </el-container>
</template>

<script>
import _ from "lodash"

const axios = require("axios").default;

export default {
  name: 'MainView',
  props: {
    msg: String
  },
  data(){
    return {
      search:{
        type: "serverjs",
        name: "",
        limit: 200
      },
      editor: {
        content: "",
        info: {
          page: null,
          time: null,
          currentPage: 1,
          count: 0
        },
        theme:{
          value: "monokai",
          list: [{ name: "亮色", items: [
                          { name: "chrome"},
                          { name: "clouds"},
                          { name: "crimson_editor"},
                          { name: "dawn"},
                          { name: "dreamweaver"},
                          { name: "eclipse"},
                          { name: "github"},
                          { name: "iplastic"},
                          { name: "solarized_light"},
                          { name: "textmate"},
                          { name: "tomorrow"},
                          { name: "xcode"},
                          { name: "kuroir"},
                          { name: "katzenmilch"},
                          { name: "sqlserver"}
                    ]
                  },
                  { name: "暗色", items: [
                          { name: "ambiance"},
                          { name: "chaos"},
                          { name: "clouds_midnight"},
                          { name: "dracula"},
                          { name: "cobalt"},
                          { name: "gruvbox"},
                          { name: "gob"},
                          { name: "idle_fingers"},
                          { name: "kr_theme"},
                          { name: "merbivore"},
                          { name: "merbivore_soft"},
                          { name: "mono_industrial"},
                          { name: "monokai"},
                          { name: "pastel_on_dark"},
                          { name: "solarized_dark"},
                          { name: "terminal"},
                          { name: "tomorrow_night"},
                          { name: "tomorrow_night_blue"},
                          { name: "tomorrow_night_bright"},
                          { name: "tomorrow_night_eighties"},
                          { name: "twilight"},
                          { name: "vibrant_ink"}
                    ]
                  }
        ]
        }
      },
      dt: {
        rows: [],
        columns: [],
        info: ""
      }
    }
  },
  components: {
    editor:require('vue2-ace-editor'),
  },
  watch:{
    'dt.rows':{
        handler(val,oldVal){
            if(val !== oldVal && !_.isEmpty(val)){
                let preFix = ['级别','时间','摘要'].join("  ");
                let csv = _.concat(preFix,_.map(val,(v)=>{
                    return `[${v.level}]  ${v.msg}`;
                })).join('\n');
                this.editor.content = csv;
            } else {
              this.editor.content = "";
            }
            
        },
        deep:true
    },
    'dt.info':{
      handler(val,oldVal){
          if(val !== oldVal && !_.isEmpty(val)){
              let info = val.split(":");
              this.editor.info.page = info[0];
              this.editor.info.time = this.moment(Number(info[1])).format('LLL');
              this.editor.info.currentPage = info[2];
              this.editor.info.count = info[3];
          } else {
            this.editor.info =  {
              page: null,
              time: null,
              currentPage: 1,
              count: 0
            };
          }
          
      }
    }
  },
  created(){
    this.initSession();
  },
  methods:{
    initSession() {
      axios
        .get(`${window.base}/user/signin?${window.signin}`)
        .then((rtn) => {
          window.session = rtn.data.message;
        })
        .catch((error) => {
          console.log(error);
        })
        .then(() => {

        });
    },
    initGlobal() {
      axios
        .get(`${window.base}/user/signin?${window.signin}`)
        .then((rtn) => {
          window.session = rtn.data.message;
        })
        .catch((error) => {
          console.log(error);
        })
        .then(() => {

        });
    },
    editorInit() {
          require('brace/ext/language_tools') //language extension prerequsite...
          require('brace/mode/html')   
          require('brace/mode/toml')                
          require('brace/mode/javascript')    //language
          require('brace/mode/less')
          require('brace/theme/chrome')
          require('brace/snippets/javascript') //snippet
    },
    onSearch(){

      axios
        .get(`${window.base}/consolelog/${this.search.type}?name=${encodeURIComponent( this.search.name.replace(/\/script/g,"") )}&limit=${this.search.limit}&sessionid=${window.session}`)
        .then((rtn) => {
          this.dt.rows = rtn.data.message.logs;
          this.dt.info = rtn.data.message.position;
        })
        .catch((error) => {
          console.error(error);
        })
        .then(() => {

        });
    },
    onCommand(val){
      console.log(val)
      require(`brace/theme/${val}`);
      this.editor.theme.value = val;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  
  .el-header{
    line-height: 60px;
  }
  
  .el-main{
    margin: 0 20px;
    padding: 0px;
  }

  .el-footer{
    line-height: 60px;
    margin: 0 20px;
    background: #f2f2f2;
    text-align: left;
  }

  .editor-toolbar{
    padding: 0 0 10px 0;
    float: left;
    width: 100%;
    height: 50px;
    line-height: 50px;
    display: flex;
  }

  .editor-toolbar > .button-group{
    width: 90%;
    text-align: left;
  }

  .el-button-group + .el-button-group{
    padding-left: 10px;
  }

  .editor-toolbar > .el-dropdown{
    text-align: right;
    width: 9%;
    margin-right: 10px;
  }

  .el-input-group__prepend > .el-select{
    width: 120px;
  }
</style>
