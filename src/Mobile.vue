<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 40,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* Inspired by http://strml.net/ which is shared by my besty
* 大家好，我是黄艺璇
* 昨天我的闺蜜小潘同学给我分享了一个动态简历
* 当时觉得炒鸡炫酷 想学！
* 于是开始上网搜索 终于让我学会了！
* 那现在开始正式编写简历吧！
*/

/* 首先给所有元素加上过渡效果 */
* {
  transition: all .3s;
}
/* 白色背景太单调了，我们来点背景 */
html {
  color: rgb(222,222,222);
  background: rgb(0,43,54);
}
/* 文字离边框太近了 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  overflow: auto;
  width: 90vw;
  margin: 2.5vh 5vw;
  height: 90vh;
}
/* 太高了 */
.styleEditor {
  height: 45vh;
}
/* 代码高亮 */
.token.selector{
  color: rgb(133,153,0);
}
.token.property{
  color: rgb(187,137,0);
}
.token.punctuation{
  color: yellow;
}
.token.function{
  color: rgb(42,161,152);
}

/* 加点 3D 效果呗 */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  transform: rotateX(-10deg) translateZ(-50px) ;
}

/* 接下来我给自己准备一个编辑器 */
.resumeEditor{
  position: fixed;
  top: 50%; left: 0;
  padding: .5em;  margin: 2.5vh;
  width: 95vw; height: 45vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 好了，我开始写简历了 */


`,
          `
/* 这个简历好像差点什么
 * 对了，这是 Markdown 格式的，我需要变成对 HR 更友好的格式
 * 简单，用开源工具翻译成 HTML 就行了
 */
`
          ,
          `
/* 再对 HTML 加点样式 */
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`],
        currentMarkdown: '',
        fullMarkdown: `黄艺璇
----
中南财经政法大学，主修税收学辅修商务英语，现在是一名大三学生。

校园经历
----

* 2017.09-2018.09 税务1501班学习委员
* 2017.09-2018.06 辅导员助理
* 2017.05-2018.05 吉他协会副会长兼团支书
* 2016.09-2017.09 税务1501班班长
* 2016.09-2017.09 院女篮篮球队队长
* 2016.09-2017.05 学院助教

实习经历
----

2018.1.8-2018.3.2        北京中小企业信用再担保有限公司   审计实习生
1.  负责制作公司2018年内控工作底稿，主要负责对子公司进行工会经费审计和协助进行对子公司的工资薪金审计。
2.  学习大量审计理论与实践知识，熟悉财务软件
2017.07-2017.09          环胜咨询服务（武汉）有限公司     财务实习生
1.	7.4-7.14  在VAT组完成每日需认证的增值税进项专票，熟悉多种财务软件
2.	7.17-8.11 负责整理自公司成立以来所有增值税进项专票抵扣联，因表现优异在（实习生）员工大会上受到嘉奖（7/80）
3.	8.14-9.1  因表现优异调至发票管理组，与会计对接每日所需录入的发票并辅助建立与测试公司数据库软件

获奖经历
----

* 2016.05   大学⽣创业创新训练计划项目国家级立项并结项      2016.11   二等人民奖学金
* 2017.03   院级寒假社会实践一等奖（主持人）               2017.04   校散打比赛女子甲组52公斤以下级”顽强拼搏奖”
* 2017.05   公共管理学院“挫折挑战赛”二等奖                 2017.05   校级优秀学生干部及院级“先锋团员”
* 2017.10   二等人民奖学金                                2017.12   全国大学生数学竞赛湖北赛区二等奖
* 2017.12   财政税务学院“公益案例分析大赛”一等奖            2018.03   财政税务学院“税收案例分析大赛”一等奖

技能及其他
----

* 英语能力：英语六级523分                            
* IT技能： 熟练运用OFFICE（特别是Excel）、VBA、财务软件
* 爱好：吉他、篮球、数独、新媒体运营、厨艺、旅游、摄影

`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          this.$nextTick(() => {
            this.$refs.resumeEditor.goTop()
          })
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    min-height: 100vh; position: relative;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }

</style>
