<template>
    <div class="category">
      <div class="category-label">
        <span v-on:click="changeFold">{{ label }}</span>
        <a v-on:click="changeFold"> {{ foldStatus }}</a>
      </div>
      <word v-bind:class="{ 'hide' : isHide }" v-bind:data="word" v-bind:idx="idx" v-for="word in data.words" v-bind:key="word.id" v-on:loadData="loadData"></word>
    </div>
</template>

<script>
import Word from './Word.vue'

export default {
  name: 'category',
  components: {
    word: Word
  },
  data () {
    return {
      isHide: true
    }
  },
  props: [
    'idx',
    'data'
  ],
  methods: {
    changeFold () {
      this.isHide = !this.isHide
    },
    loadData (id) {
      this.$emit('loadData', this.idx, id)
    }
  },
  computed: {
    label () {
      return this.data.label
    },
    foldStatus () {
      return this.isHide ? '펼치기 ▼' : '접기 ◀'
    }
  }
}
</script>

<style scoped>
  .hide {
    display:none
  }
  .category {
    padding:10px 0px 0px 20px
  }
  .category-label > span{
    font-size: 15pt;
    font-weight: bold;
    color:#ffffaa;
    cursor:pointer;
  }

  .category-label > a{
    color: #ffffff;
    float:right;
    cursor: pointer;
    font-family: "맑은 고딕";
    font-size: 9pt;
    font-weight: bold;
    letter-spacing: -1px;
    padding-right: 5px
  }
</style>
