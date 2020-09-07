<template>
  <div class="scene">
    <div id="ctrl"> 
      <button @click="move('z', -5)"> 前 </button> 
      <button @click="move('z', 5)"> 後 </button>
      <button @click="move('x', -5)"> 左 </button> 
      <button @click="move('x', 5)"> 右 </button>
    </div>
    <div id="win" v-show="win" @click="win = false">
      <img src="../assets/rainbow.jpg">
    </div>
    <div id="three-scene-canvas"></div>
  </div>
  
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

export default {
  name: 'Scene',
  data () {
    return {
      sceneCanvas: null,
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      meshArray: [],
      me: null,
      you: null,
      win: false,
      audio: null
    }
  },
  created () {
    this.audio = document.getElementById('audio');
  },
  mounted () {
    /* **************
       BASIC SETUP
    ************** */

    this.sceneCanvas = document.getElementById('three-scene-canvas')
    this.scene = new THREE.Scene()
    this.camera = new THREE.PerspectiveCamera(
      75,
      this.sceneCanvas.getBoundingClientRect().width / this.sceneCanvas.getBoundingClientRect().height,
      0.1,
      1000
    )
    this.camera.position.set(20, 5, -20)
    
    this.renderer = new THREE.WebGLRenderer({
      antialias: true,
      powerPreference: "high-performance"
    })
    this.renderer.outputEncoding = THREE.sRGBEncoding

    this.controls = new OrbitControls(this.camera, this.renderer.domElement)
    this.controls.addEventListener('change', this.animateThreeJs )

    this.renderer.setSize(this.sceneCanvas.offsetWidth, this.sceneCanvas.offsetHeight)
    this.renderer.setClearColor("#212121")
    this.renderer.shadowMap.enabled = true
    this.renderer.shadowMap.type = THREE.PCFSoftShadowMap
    this.renderer.shadowMapSoft = true
    this.renderer.shadowMap.autoUpdate = false
    this.renderer.shadowMap.needsUpdate = true
    this.sceneCanvas.append(this.renderer.domElement)
    
    // lighting
    let ambientLight = new THREE.AmbientLight (0xdaccff, 0.5)
    this.scene.add(ambientLight)

    let light = new THREE.PointLight(0xfc831d, 1, 100)
    light.position.set(15, 10, 15)
    light.castShadow = true
    light.shadow.radius = 1
    light.shadow.mapSize.width = 2048
    light.shadow.mapSize.height = 2048
    this.scene.add(light)

    // Adding cubes
    let material = new THREE.MeshPhysicalMaterial({color: 0x00ff00})
    var BoxGeometry = new THREE.BoxGeometry(5, 2, 5)
    var maze = [
      [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
      [1,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,1],
      [1,0,1,1,1,0,1,1,1,0,1,1,1,5,1,0,1,1,1,1],
      [1,0,0,0,1,1,1,0,1,0,1,1,1,0,1,0,1,1,0,1],
      [1,0,1,0,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,1],
      [1,0,1,1,1,0,1,0,1,0,1,1,1,1,1,0,1,1,0,1],
      [1,0,1,0,1,0,1,0,1,0,1,1,0,0,1,0,1,1,0,1],
      [1,0,1,0,1,0,1,0,1,0,0,0,0,1,1,0,1,1,0,1],
      [1,0,0,0,1,0,1,0,1,1,1,1,0,1,1,0,1,1,0,1],
      [1,0,1,1,1,0,1,0,1,0,1,1,0,1,1,0,1,1,0,1],
      [1,0,1,1,1,0,1,0,1,0,1,0,0,0,1,0,1,1,0,1],
      [1,0,0,0,1,1,1,0,1,0,1,1,1,0,1,0,1,1,0,1],
      [1,1,1,0,1,0,0,0,1,0,0,0,0,0,1,0,1,1,0,1],
      [1,0,0,0,1,1,0,1,1,0,1,1,1,0,1,0,1,1,0,1],
      [1,1,1,0,1,0,0,0,0,0,0,0,1,0,1,0,1,1,0,1],
      [1,0,0,0,1,1,0,1,1,0,1,1,1,0,1,0,1,1,0,1],
      [1,0,1,1,1,1,0,1,1,0,1,1,1,0,1,0,1,1,0,1],
      [1,0,1,9,1,1,0,1,1,0,1,1,1,0,1,0,1,1,0,1],
      [1,0,1,0,1,1,0,1,1,0,1,1,1,0,1,0,1,1,0,1],
      [1,0,0,0,1,0,0,0,0,0,1,1,1,0,0,0,0,0,0,1],
      [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
    ];
    this.meshArray = [];
    for (var x=0; x<maze.length; x++) {
        for(var z=0; z<maze[0].length; z++){
          var mesh;
          if (maze[x][z] === 1) {
            mesh = new THREE.Mesh(BoxGeometry, material.clone());
            this.meshArray.push(mesh);
            mesh.position.x = -5*x - 10;
            mesh.position.y = -20;
            mesh.position.z = -5*z + 20;
            this.scene.add(mesh);
          }
          if (maze[x][z] === 9) {
            var Spheregeometry = new THREE.SphereGeometry( 1, 32, 32 );
            var material2 = new THREE.MeshBasicMaterial( {color: 0xffff00} );
            this.you = new THREE.Mesh( Spheregeometry, material2.clone() );
            this.you.position.x = -5*x - 10;
            this.you.position.y = -20;
            this.you.position.z = -5*z + 20;
            this.scene.add(this.you);  
          }
          if (maze[x][z] === 5) {
            var Spheregeometry2 = new THREE.SphereGeometry( 1, 32, 32 );
            var material3 = new THREE.MeshBasicMaterial( {color: 0x00ff} );
            this.me = new THREE.Mesh( Spheregeometry2, material3.clone() );
            this.me.position.x = -5*x - 10;
            this.me.position.y = -20;
            this.me.position.z = -5*z + 20;
            this.scene.add(this.me);  
          }
        }
    }
    this.animateThreeJs();
    window.addEventListener('keyup', this.keyup);
  },
  methods: {
    animateThreeJs () {
      this.renderer.render(this.scene, this.camera)
      this.renderer.shadowMap.needsUpdate = true
    },
    keyup(e) {
      if (e.which === 37) { this.move('x', -5) }
      if (e.which === 38) { this.move('z', -5) }
      if (e.which === 39) { this.move('x', 5) }
      if (e.which === 40) { this.move('z', 5) }
    },
    move (xyz, int) {
      this.me.position[xyz] += int;
      for (var i = 0; i < this.meshArray.length; i++) {

        var position = new THREE.Vector3();
        position.getPositionFromMatrix( this.meshArray[i].matrixWorld );

        if (position.x === this.me.position.x && position.z === this.me.position.z ) {
          this.me.position[xyz] -= int;
        }

        var xz = {x: this.me.position.x, z: this.me.position.z }
        
        this.camera.position.set(xz.x, this.me.position.y + 20, xz.z);
        this.controls.target.set(this.me.position.x, this.me.position.y, this.me.position.z);
        this.controls.update();
      }
      if (this.me.position.x === this.you.position.x && this.me.position.z === this.you.position.z ) {
        this.win = true
      }
      this.animateThreeJs();
    }
  }
}
</script>

<style type="text/css" scoped="">

#ctrl {
  position: fixed;
  bottom: 15px;
  left: 0;
  z-index: 999;
}

#ctrl button {
  font-size: 48px;
}

#win {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 50px;
  left: 50px;
  width: 80vw;
  height: 80vh;
  cursor: pointer;
}

#win img {
  width: 100%
}

</style>