<template>
  <div class="scene">
    <div id="ctrl"> 
      <button id="up" @click="move('z', -5); m='up'" :class = "{active: m == 'up' }"> 前 </button> 
      <button id="down" @click="move('z', 5); m='down'" :class = "{ active: m == 'down' }"> 後 </button>
      <button id="left" @click="move('x', -5); m='left'" :class = "{ active: m == 'left' }"> 左 </button> 
      <button id="right" @click="move('x', 5); m='right'" :class = "{ active: m == 'right' }"> 右 </button>
    </div>
    <a id="win" v-show="win" @click="win = false">
      <img src="../assets/rainbow.jpg">
    </a>
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
      audio: null,
      m: null,
      lev: 0,
    // 自訂迷宮
    // 0: 空地
    // 1: 牆
    // 5: 起點
    // 9: 終點
      maze: [
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
      ]
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

    this.meshArray = [];
    for (var x=0; x < this.maze.length; x++) {
        for(var z=0; z <this.maze[0].length; z++){
          var mesh;
          // 牆壁
          if (this.maze[x][z] === 1) {
            let c = [0x3a406e, 0x00a6ff, 0xff003b][Math.floor(Math.random()*3)]
            let material = new THREE.MeshPhysicalMaterial({color: c})
            let BoxGeometry = new THREE.BoxGeometry(5, 2, 5)
            mesh = new THREE.Mesh(BoxGeometry, material.clone());
            this.meshArray.push(mesh);
            mesh.position.x = -5*x - 10;
            mesh.position.y = -20;
            mesh.position.z = -5*z + 20;
            this.scene.add(mesh);
          }

          // 終點
          if (this.maze[x][z] === 9) {
            let BoxGeometry = new THREE.BoxGeometry( 2, 30, 2);
            let material2 = new THREE.MeshBasicMaterial( {color: 0xffff00} );
            this.you = new THREE.Mesh( BoxGeometry, material2.clone() );
            this.you.position.x = -5*x - 10;
            this.you.position.y = -20;
            this.you.position.z = -5*z + 20;
            this.scene.add(this.you);  
          }

          // 起點
          if (this.maze[x][z] === 5) {
            let Spheregeometry = new THREE.SphereGeometry( 1, 32, 32 );
            let material3 = new THREE.MeshBasicMaterial( {color: 0x00ff} );
            this.me = new THREE.Mesh( Spheregeometry, material3.clone() );
            this.me.position.x = -5*x - 10;
            this.me.position.y = -20;
            this.me.position.z = -5*z + 20;
            this.scene.add(this.me);  
          }
        }
    }
    this.animateThreeJs();
    window.addEventListener('keyup', this.keyup); // 聽鍵盤事件
  },
  methods: {
    animateThreeJs () {
      this.renderer.render(this.scene, this.camera)
      this.renderer.shadowMap.needsUpdate = true
    },
    keyup(e) {
      // left key
      if (e.which === 37) { this.move('x', -5); this.m = 'left' }
      // up key
      if (e.which === 38) { this.move('z', -5); this.m = 'up' }
      // right key
      if (e.which === 39) { this.move('x', 5); this.m = 'right' }
      // down key
      if (e.which === 40) { this.move('z', 5); this.m = 'down' }
      // a key
      if (e.which === 65) { this.move('x', -5); this.m = 'left' }
      // w key
      if (e.which === 87) { this.move('z', -5); this.m = 'up' }
      // d key
      if (e.which === 68) { this.move('x', 5); this.m = 'right' }
      // s key
      if (e.which === 83) { this.move('z', 5); this.m = 'down' }
    },
    move (xyz, int) {
      this.me.position[xyz] += int;
      for (var i = 0; i < this.meshArray.length; i++) {

        var position = new THREE.Vector3();
        position.getPositionFromMatrix( this.meshArray[i].matrixWorld );

        if (position.x === this.me.position.x && position.z === this.me.position.z ) {
          this.me.position[xyz] -= int;
        } // 如果撞牆就取消移動

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
  bottom: 150px;
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
  width: 50vw;
  height: 50vh;
  cursor: pointer;
}

#win img {
  cursor: pointer;
  width: 100%;
}

#left, #right, #up, #down {
  position: absolute;
  border-radius: 50%;
  cursor: pointer;
}

.active {
  background-color: blue;
}

#left, #right {
  top: 0px;
}

#up, #down {
  left: 50px;
}

#up {
  top: -50px;
}

#down {
  top: 50px;
}

#left {
  left: 0
}

#right {
  left: 100px;
}


</style>