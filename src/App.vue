<template>
  <div id="app">
    <aplayer autoplay :repeat="'repeat-one'"
    :music="{
      title: 'Background',
      artist: 'Game',
      src: '/pokemon.mp3'
    }"
    />
    <Scene/>
  </div>
</template>

<script>
import Scene from './components/Scene.vue'
import Aplayer from 'vue-aplayer'

export default {
  name: 'App',
  components: {
    Scene, Aplayer
  }
}
</script>

<style lang="scss">
@import '/assets/style/custom.scss';
</style>