<script setup>
  import Row from './Row.vue'
  defineProps({
    rows: Array
  })
</script>

<template>
  <div class="keyboard">
    <Row :letters="row" v-for="row in rows" :keyboard="true"></Row>
  </div>
</template>

<style>
  .keyboard {
    position: absolute;
    bottom: 2rem;
    left: 0;
    right: 0;
    margin: auto
  }
  .keyboard .tile {
    cursor: pointer;
  }
  @media (max-width: 766px){
    .keyboard .tile {
      width:  2.5rem;
      height: 2.5rem;
      font-size: 1.5rem;
    }
    .keyboard .tile.wide {
      width: 5.5rem;
    }
    .keyboard .dot.tr {
      margin-top: 1.1rem;
      margin-left: 1.2rem;
    }
    .keyboard .dot.ml {
      margin-top: 1rem;
      margin-left: -1.5rem;
    }
    .keyboard .dot.mc {
      margin-top: -1.3rem;
      margin-left: -1.7rem;
    }
    .keyboard .dot.lr {
    margin-top: -1.5rem;
    margin-left: 1.5rem;
  }
  }

  @media (max-width: 500px){
    .keyboard .tile {
      width:  1.5rem;
      height: 1.5rem;
      font-size: 1.25rem;
    }
    .keyboard .tile.wide {
      width: 4rem;
    }

  }
  @media (max-width: 400px){
    .keyboard .tile {
      width:  1rem;
      height: 1.5rem;
      font-size: .75rem;
    }
    .keyboard .tile.wide {
      width: 2.25rem;
    }

  }
</style>