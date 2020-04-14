<template>
  <div id="app">    
    <chessboard></chessboard>
  </div>
</template>

<script>
import Chessboard from "@/components/Chessboard.vue";

export default {
  name: 'App',
  components: {
    Chessboard
  },
  data() {
    return {
      name: 'test'
    }
  }
}
</script>

<style>

</style>
