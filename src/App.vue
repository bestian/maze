<template>
  <div id="app">
    <aplayer autoplay mini :repeat="'repeat-one'"
    :music="{
      title: 'Background',
      artist: 'Game',
      src: 'https://bestian.github.io/maze/happy.mp3'
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
  },
  mounted () {
    console.log('hello')
    var elements = document.getElementsByClassName('aplayer-icon')
    for (var index = 0; index < elements.length; index++) {
      elements[index].setAttribute('name', 'Play')
      elements[index].setAttribute('aria-label', 'Play')
    }
  }
}
</script>

<style lang="scss">
@import '/assets/style/custom.scss';
</style>