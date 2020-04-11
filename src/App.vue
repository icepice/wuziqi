<template>
  <div id="app">    
    <chess-board></chess-board>
  </div>
</template>

<script>
import ChessBoard from "@/components/ChessBoard.vue";

export default {
  name: 'App',
  components: {
    ChessBoard
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
