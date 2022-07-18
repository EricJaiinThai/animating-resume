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
* Inspired by http://strml.net/
* สวัสดีผมอีริค จัย ถึงฤดูการเปลี่ยนงานแล้ว
* หลายคนกำลังทำเรซูเม่สมัครงานอยู่
* ผมเองก็เช่นกัน
*/

/* ขั้นแรกให้เพิ่มเอฟเฟกต์การเปลี่ยนภาพให้กับองค์ประกอบทั้งหมด */
* {
  transition: all .3s;
}
/* พื้นหลังสีขาวมันน่าเบื่อเกินไป เรามาทำพื้นหลังใหม่กันเถอะ */
html {
  color: rgb(222,222,222); background: rgb(0,43,54);
}
/* ข้อความก็ชิดขอบเกินไป */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 45vw; height: 90vh;
}
/* เติมสีสันให้โค้ดสักหน่อย */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* ทำให้มันดูสามมิติสักหน่อย */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  -webkit-transition: none;
  transition: none;
  -webkit-transform: rotateY(10deg) translateZ(-100px) ;
          transform: rotateY(10deg) translateZ(-100px) ;
}

/* แล้วเรามาเพิ่มรายละเอียดกันหน่อย */
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  width: 48vw; height: 90vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* เอาล่ะ ฉันเริ่มเขียนเรซูเม่แล้วน่ะ */


`,
          `
/* เรซูเม่นี้ดูเรียบง่ายเกินไป
 * อีกอย่าง มันอยู่ในรูปแบบ Markdown ฉันต้องเปลี่ยนเป็นรูปแบบที่เป็นมิตรกับ HR มากขึ้น
 * ไม่ยาก แค่เพียงแปลเป็น HTML ด้วยเครื่องมือโอเพ่นซอร์ส
 */
`
          ,
          `
/* ทำ HTML ให้มีรสนิยมสักหน่อย */
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
        fullMarkdown: `方应杭
----

ปัจจุบันเป็นพนักงานธุรกิจสินเชื่อ (ธ.ก.ส.)，รับผิดชอบงานวิเคราะห์สินเชื่อรายใหญ่。

ทักษะ
----

* การรวบรวมข้อมูลเพื่อนำไปวิเคราะห์
* การย่อยข้อมูลเพื่อวิเคราะห์การทำงานของระบบ
* การกลั่นกรองข้อมูลที่นำมาวิเคราะห์
* การเลือกใช้เฟรมเวิร์คในการวิเคราะห์ให้เหมาะสมกับการใช้งาน

ประสบการณ์การทำงาน
----

1. ผู้ช่วยผู้จัดการสำนักงานบัญชี ภูเก็ต อินเตอร์เนชั่นแนล แอคเคานท์ติ้ง แอนด์ บิสซิเนส คอนซัลติ้ง เฟิร์ม
2. เจ้าหน้าที่ธุรการหน้างาน (โครงการโรบินสันปราจีนบุรี) บริษัท เอ็ม วี เอส ดีเวลลอปเม้นท์ 1688 จำกัด
3. เจ้าหน้าที่การตลาด บริษัท ไดมอนด์ ฟู้ด โปรดักท์ จำกัด
4. พนักงานธุรกิจสินเชื่อ ธนาคารเพื่อการเกษตรและสหกรณ์การเกษตร

ลิงค์
----

* [GitHub](https://github.com/frankfang)
* [我的文章](https://www.zhihu.com/people/zhihusucks/pins/posts)

> ถ้าคุณชอบเอฟเฟกต์นี้，Fork [我的项目](https://github.com/jirengu-inc/animating-resume)，สร้างประวัติย่อของคุณเอง!

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
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }
</style>
