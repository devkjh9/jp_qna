<template>
  <div id="app">
    <aside>
      <div class="menu-title">메뉴</div>
      <category v-bind:data="category" v-for="(category, index) in categories" v-bind:key="'category' + index" v-bind:idx="index" v-on:loadData="loadData"></category>
    </aside>
    <header>
      <h3>일본어 문제</h3>
    </header>
    <article>
      <div class="div-question-area">
        <h3 class="h3-question-title">{{ getTitle }}</h3>
        <div class="div-question-wrapper">
          <div class="div-question">{{ getQuestion }}</div>
          <div class="div-question-answer-wrapper" v-if="isShowAnswer">
            <div class="div-question-answer" v-for="(answers, index) in getAnswer" v-bind:key="'answer' + index">
            {{ answers }}
            </div>
          </div>
        </div>
        <div class="div-question-status">
          문제 수 : {{ getQuestionCount }}
          / 정답 수 : {{ getRightCount }} ({{ getRightPercent }}%)
          / 남은 수 : {{ getIncompletedCount }}
        </div>
        <div class="div-question-status-control">
          <a class="a-question-status-wrong-cancel" v-on:click="cancelWrong">[오답만 일괄 문제 회수]</a>
          <a class="a-question-status-hide" v-on:click="switchShowCompletedQuestion">{{ isShowCompletedQuestion ? '[완료 문제 숨기기]' : '[완료 문제 보이기]' }}</a>
        </div>
        <div class="div-question-completed-wrapper" v-if="isShowCompletedQuestion">
          <span class="span-question-completed"
                v-for="(question, index) in completed"
                v-bind:key="'completed' + index"
                v-bind:style="{ color : question.isRight ? '#0000ff' : '#ffaa66' }"
                v-on:click="cancelCompleteQuestion(index)"
          >
            {{ question.question }}
          </span>
        </div>
      </div>
      <div class="div-answer-area">
        <button class="btn-info" v-on:click="showAnswer">확인</button>
        <button class="btn-primary" v-on:click="answer(true)">정답</button>
        <button class="btn-danger" v-on:click="answer(false)">오답</button>
        <button class="btn-success" v-on:click="next">스킵</button>
        <ccanvas />
      </div>
    </article>
  </div>
</template>

<script>
import words from './words/words'
import Category from './components/Category.vue'
import CCanvas from './components/CCanvas.vue'

const cookieName = 'save'

export default {
  name: 'app',
  components: {
    'category': Category, 'ccanvas': CCanvas
  },
  data () {
    return {
      categories: words.categories,
      categoryIdx: -1,
      wordId: null,
      title: '불러온 데이터 없음',
      question: [],
      wait: [],
      completed: [],
      current: -1,
      isShowAnswer: false,
      isShowCompletedQuestion: false
    }
  },
  methods: {
    loadData (idx, id, isCookie) {
      const item = this.categories[idx].words.filter(c => c.id === id)[0]
      const data = item
        .data
        .map((v, i) => {
          v.index = i
          return v
        })
      this.categoryIdx = idx
      this.wordId = id
      this.title = item.name
      this.question = data.slice()
      this.wait = data.slice()
      this.completed = []
      this.current = -1
      if (isCookie === undefined) {
        this.next()
      }
    },
    next () {
      if (this.wait.length > 0) {
        this.isShowAnswer = false
        this.current = Math.floor(Math.random() * this.wait.length)
        this.$emit('clearCanvas')
      }
    },
    answer (isRight) {
      if (this.current > -1) {
        const current = this.current
        this.current = -1
        const item = this.wait[current]
        item.isRight = isRight
        this.completed.push(item)
        this.wait.splice(current, 1)
        this.saveCookie()
        this.next()
      }
    },
    showAnswer () {
      if (this.current > -1 && this.isShowAnswer !== true) this.isShowAnswer = true
    },
    switchShowCompletedQuestion () {
      this.isShowCompletedQuestion = !this.isShowCompletedQuestion
    },
    cancelCompleteQuestion (index) {
      this.wait.push(this.completed[index])
      this.completed.splice(index, 1)
      this.saveCookie()
      if (this.current < 0) {
        this.next()
      }
    },
    cancelWrong () {
      const wrong = this.completed.filter(c => c.isRight === false)
      if (wrong.length > 0) {
        this.wait.push(...wrong)
        this.completed = this.completed.filter(c => c.isRight === true)
        this.saveCookie()
        if (this.current < 0) {
          this.next()
        }
      }
    },
    confirmLoad () {
      this.$cookies.remove('japQna2-cookie')
      if (this.$cookies.isKey(cookieName)) {
        if (confirm('저장된 이력이 있습니다.\n다시 불러오시겠습니까?')) {
          const cookie = this.$cookies.get(cookieName)
          if (cookie && cookie.x !== undefined &&
            cookie.w !== undefined && cookie.c !== undefined) {
            this.loadData(cookie.x, cookie.w, true)
            cookie.c.forEach(v => {
              const q = this.question[v.i]
              const data = { isRight: !!(v.r === 1) }
              for (let i in q) {
                data[i] = q[i]
              }
              this.completed.push(data)
              this.wait = this.wait.filter(w => w.index !== v.i)
            })
            this.next()
          } else {
            this.$cookies.remove(cookieName)
          }
        } else {
          this.$cookies.remove(cookieName)
        }
      }
    },
    saveCookie () {
      const data = {
        x: this.categoryIdx,
        w: this.wordId,
        c: this.completed.map(v => {
          return { i: v.index, r: (v.isRight ? 1 : 0) }
        })
      }
      this.$cookies.set(cookieName, data)
    }
  },
  computed: {
    getTitle () {
      return this.title
    },
    getQuestionCount () {
      return this.question.length
    },
    getRightCount () {
      return this.completed.filter(c => c.isRight === true).length
    },
    getRightPercent () {
      if (this.question.length > 0) {
        return Math.floor(this.completed.filter(c => c.isRight === true).length * 100 / this.question.length)
      } else {
        return 0
      }
    },
    getIncompletedCount () {
      return this.wait.length
    },
    getQuestion () {
      if (this.current > -1) {
        return this.wait[this.current].question
      } else if (this.question.length > 0 && this.wait.length <= 0) {
        return '문제 풀이 완료'
      } else {
        return ''
      }
    },
    getAnswer () {
      if (this.current > -1) {
        return this.wait[this.current].answer
      } else {
        return []
      }
    }
  },
  mounted () {
    const that = this
    const body = document.body
    body.addEventListener('keydown', function (e) {
      switch (e.keyCode) {
        case 49: that.showAnswer()
          break
        case 50: that.answer(true)
          break
        case 51: that.answer(false)
          break
      }
    })
    this.confirmLoad()
  }
}
</script>

<style>
  @import url('https://fonts.googleapis.com/earlyaccess/nanummyeongjo.css');

  html, head, body, section, nav, h1, h2, h3, h4, h5, h6, ul, li {
    padding: 0;
    margin: 0
  }
  html {
    background: #fafafa;
    font-family: "Nanum Myeongjo";
  }
  body {
    padding-bottom: 15px;
    text-align: center;
  }
  aside{
    background: #888;
    padding: 90px 20px 30px 0px;
    position: absolute;
    width: 303px;
    margin: auto;
    z-index: 1000;
    text-align: left;
  }
  header{
    background:#99d999;
    padding: 15px 0px 15px 20px;
  }
  article {
    padding-left: 25%
  }
  #app {
    background: #fafafa;
  }
  .menu-title {
    padding: 10px;
    width: 70%;
    background: #ddffdd;
    font-weight: bold;
  }
  button {
    border: 1px solid transparent;
    border-radius: 4px;
    padding: 9px 40px;
    cursor:pointer;
    color: #ffffff;
    font-weight: bold;
    font-size: 12pt
  }
  .btn-primary {
    background-color: #337ab7;
    border-color: #2e6da4;
  }
  .btn-primary:hover {
    background-color: #286090;
    border-color: #204d74;
  }
  .btn-success {
    background-color: #5cb85c;
    border-color: #4cae4c;
  }
  .btn-success:hover {
    background-color: #449d44;
    border-color: #398439;
  }
  .btn-info {
    background-color: #5bc0de;
    border-color: #46b8da;
  }
  .btn-info:hover {
    background-color: #31b0d5;
    border-color: #269abc;
  }
  .btn-danger {
    background-color: #d9534f;
    border-color: #d43f3a;
  }
  .btn-danger:hover {
    background-color: #c9302c;
    border-color: #ac2925;
  }
  .h3-question-title {
    padding: 3px 0px;
    text-align: center;
  }
  .div-question-area {
    float: left;
  }
  .div-question-wrapper{
    width: 480px;
    height: 480px;
    text-align: center;
    background: #ffffff;
    display: table;
    position: relative;
    margin: auto
  }
  .div-question{
    width: 100%;
    height: 100%;
    vertical-align: middle;
    display: table-cell;
    font-family: Meiryo;
    font-size: 48pt
  }
  .div-question-answer-wrapper{
    position: absolute;
    z-index: 999;
    top: 10%;
    left: 10%;
    max-width: 80%;
  }
  .div-question-answer{
    margin-top: 40px;
    font-size: 16pt;
    letter-spacing: -1px;
    border-radius: 3px;
    padding: 5px 15px;
    background: rgba(153, 217, 153, 0.4);
    color: #666666;
    font-family: Meiryo;
    word-break: break-all;
  }
  .div-question-answer:hover{
    background: rgba(153, 217, 153, 1);
  }
  .div-question-answer:first-child{
    margin-top: 0px;
  }
  .div-question-status {
    padding: 3px 0px
  }
  .div-question-status-control{
    font-size: 11pt;
    padding: 2px 0px;
  }
  .div-question-status-control > a {
    cursor:pointer;
  }
  .div-question-status-control > a:hover{
    text-decoration: underline;
  }
  .a-question-status-wrong-cancel{
    color: #ffaa66
  }
  .a-question-status-hide {
    color: #0000ff
  }
  .div-question-completed-wrapper{
    max-width: 480px;
    margin:auto;
  }
  .span-question-completed {
    cursor: pointer;
  }
  .span-question-completed:not(:first-child):before{
    content: '|';
    display: inline-block;
    margin-right:5px;
    font-size:8pt;
    color: #333333;
    line-height: 21px;
  }
  .div-answer-area{
    float: left;
    margin: 27px 0px 15px 5px
  }
  .div-answer-area > button{
    margin: 0 2px
  }

  @media screen and (max-width: 1469px) {
    aside {
      padding: 0px 0px 30px 0px;
      position: static;
      width: 100%;
      margin: 0;
    }
    article {
      padding-left: 0
    }
    .div-question-area {
      float: none;
    }
    .div-answer-area > button{
      width: 23%;
      padding: 9px 25px
    }
    .div-question-wrapper {
      width: 100%;
      height: 300px;
    }
    .div-question-answer-wrapper{
      max-height: 90%;
      overflow-y: auto;
    }
    .div-question-answer{
    }
    .div-answer-area{
      float: none;
      margin: 15px 0px 15px 0px
    }
  }
</style>
