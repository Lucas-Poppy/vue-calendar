<template>
  <div class="container" ref="container">
    <div v-for="l in list" :key="l">{{hoge(l)}}</div>
    <button @click="hashChange" >ハッシュ変更！</button>
  </div>
</template>

<script>
export default {
  data () {
    return {
      lists: [
        1,
        2,
        3,
        4,
        5
      ],
      hash: {
        one: 1,
        two: 2,
        three: 2
      }
    }
  },
  methods: {
    hashChange () {
      this.$set(this.hash, 'four', 4)
    },
    hoge (list) {
      console.log('hoge')
      return list
    }
  },
  computed: {
    list () {
      return this.lists
    }
  }
}
</script>
