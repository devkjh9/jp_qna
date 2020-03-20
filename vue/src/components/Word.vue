<template>
    <div class="word">
      <a v-on:click="loadData">{{ data.name }}</a>
    </div>
</template>

<script>
export default {
  name: 'Word',
  props: [
    'idx',
    'data'
  ],
  methods: {
    loadData () {
      this.$emit('loadData', this.data.id)
    }
  }
}
</script>

<style scoped>
  .word {
    padding-left:20px;
    color: #ffffff;
  }
  .word > a {
    cursor: pointer;
  }
  .word > a:hover {
    color: #cccc00
  }
</style>
