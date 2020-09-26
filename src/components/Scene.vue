<template>
  <div class="scene">
    <div id="ctrl"> 
      <button id="up" @click="move('z', -5); m='up'" :class = "{active: m == 'up' }"> 前 </button> 
      <button id="down" @click="move('z', 5); m='down'" :class = "{ active: m == 'down' }"> 後 </button>
      <button id="left" @click="move('x', -5); m='left'" :class = "{ active: m == 'left' }"> 左 </button> 
      <button id="right" @click="move('x', 5); m='right'" :class = "{ active: m == 'right' }"> 右 </button>
    </div>
    <div id = "lev">
        目前{{lev}}級
        <br/>
        目前食物:{{food}}
    </div>
    <a id="win" v-show="win" @click="win = false">
      <img src="../assets/cat.jpg">
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
      food: 300,
    // 自訂迷宮
    // 0: 空地
    // 1: 牆
    // 5: 起點
    // 9: 終點
      maze: null,
      mazes: ['maze0.json','maze1.json','maze2.json'],
      colors: [
        [0x3a406e, 0x00a6ff, 0xff003b],
        [0x3a6e40, 0xffa600, 0x00ff3b],
        [0x403a6e, 0xa600ff, 0x003bff]
      ],
      borders: [
        [0x6a709e, 0x30d6ff, 0xff306b],
        [0x6a9e70, 0xffd630, 0x30ff6b],
        [0x706a9e, 0xd630ff, 0x306bff]
      ]
    }
  },
  created () {
    this.audio = document.getElementById('audio');
  },
  mounted () {
    this.sceneCanvas = document.getElementById('three-scene-canvas')
    this.scene = new THREE.Scene()
    this.camera = new THREE.PerspectiveCamera(
      75,
      this.sceneCanvas.getBoundingClientRect().width / this.sceneCanvas.getBoundingClientRect().height,
      0.1,
      1000
    )    
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

    if (localStorage.lev) {
      this.lev = localStorage.lev;
    }
    if (localStorage.food) {
      this.food = localStorage.food;
    }
    if (localStorage.me) {
      this.me = localStorage.me;
    }
    this.reset(this.lev)
    window.addEventListener('keyup', this.keyup); // 聽鍵盤事件
  },
  watch: {
    lev(newLev) {
      localStorage.lev = newLev;
    },
    food(newfood) {
      localStorage.food = newfood;
    },
    me(newme) {
      localStorage.me = newme;
    }
  },
  methods: {
    reset (lev) {
      console.log(lev)

      this.camera.position.set(20, 5, -20)

      while(this.scene.children.length > 0){ 
          this.scene.remove(this.scene.children[0]); 
      }
      // lighting
      let ambientLight = new THREE.AmbientLight (0xdaccff, 0.5)
      this.scene.add(ambientLight)

      let light = new THREE.PointLight(0xfc831d, 1, 300)
      light.position.set(20, 100, -20)
      light.castShadow = true
      light.shadow.radius = 1
      this.scene.add(light)

      this.axios.get('./maze'+lev+'.json').then((response) => {
        // console.log(response.data)
        this.maze = response.data;

        this.meshArray = [];
        for (var x=0; x < this.maze.length; x++) {
            for(var z=0; z <this.maze[0].length; z++){
              var mesh;
              // 牆壁
              var c;

              let material = new THREE.MeshPhysicalMaterial({color: 0xc9c9c9})
              let BoxGeometry = new THREE.BoxGeometry(5, 2, 5);
                mesh = new THREE.Mesh(BoxGeometry, material.clone());

              mesh.position.x = -5*x - 10;
              mesh.position.y = -22;
              mesh.position.z = -5*z + 20;

              mesh.receiveShadow = true;
              this.scene.add(mesh);

              if (this.maze[x][z] === 1) {
                let r = Math.floor(Math.random()*3);
                c = this.colors[lev][r];
                let material = new THREE.MeshPhysicalMaterial({color: c})
                let BoxGeometry = new THREE.BoxGeometry(5, 2, 5);
                mesh = new THREE.Mesh(BoxGeometry, material.clone());
                this.meshArray.push(mesh);
                mesh.position.x = -5*x - 10;
                mesh.position.y = -20;
                mesh.position.z = -5*z + 20;
                mesh.castShadow = true;

                // wireframe
                c = this.borders[lev][r]
                var geo = new THREE.EdgesGeometry( mesh.geometry );
                var mat = new THREE.LineBasicMaterial( { color: c, linewidth: 4 } );
                var wireframe = new THREE.LineSegments( geo, mat );
                wireframe.renderOrder = 1; // make sure wireframes are rendered 2nd
                mesh.add( wireframe );

                this.scene.add(mesh);
              }

              // 終點
              if (this.maze[x][z] === 9) {
                let BoxGeometry = new THREE.BoxGeometry( 2, 50, 2);
                let material2 = new THREE.MeshBasicMaterial( {color: 0xffff00} );
                this.you = new THREE.Mesh( BoxGeometry, material2.clone() );
                this.you.position.x = -5*x - 10;
                this.you.position.y = 5;
                this.you.position.z = -5*z + 20;
                this.you.castShadow = true;
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
                this.me.castShadow = true;
                this.scene.add(this.me);  
              }
            }
        }
        this.animateThreeJs();
      })
    },
    animateThreeJs () {
      this.renderer.render(this.scene, this.camera)
      this.renderer.shadowMap.needsUpdate = true
    },
    victory () {
      if (this.lev == 2) {
        this.lev = 0;
        this.win = true
        //噴出很多方塊
        var x = this.you.position.x;
        var z = this.you.position.z;

        var mesh;

        for (var i = 0; i < [-4,-3,-2,-1,0,1,2,3,4].length; i++) {
          for (var j = 0; j < [-4,-3,-2,-1,0,1,2,3,4].length; j++) {
            let c = [0xcd7f33, 0xd9d9e9, 0xcfc0b3][Math.floor(Math.random()*3)]
            let material = new THREE.MeshPhysicalMaterial({color: c})
            let BoxGeometry = new THREE.BoxGeometry(5, 2, 5)
            mesh = new THREE.Mesh(BoxGeometry, material.clone());

            this.meshArray.push(mesh);
            mesh.position.x = x + 10*i - 10;
            mesh.position.y = -5;
            mesh.position.z = z - 10*j + 20;
            this.scene.add(mesh);
          }
        }
        this.camera.position.set(x, this.me.position.y + 50, z);
        this.controls.update();
        this.animateThreeJs();
      } else {
        alert('你升級啦！')
        this.lev++;
        this.reset(this.lev)
      }
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
      // v key
      if (e.which === 86) { this.die() }
    },
    die () {
      window.alert('你餓死了');
      this.lev = 0;
      this.food = 300;
      this.reset(this.lev)
    },
    move (xyz, int) {
      this.food--;
      if (this.food === 0) {
        this.die()
      }
      this.me.position[xyz] += int;
      for (var i = 0; i < this.meshArray.length; i++) {

        var position = new THREE.Vector3();
        position.setFromMatrixPosition( this.meshArray[i].matrixWorld );

        if (position.x === this.me.position.x && position.z === this.me.position.z ) {
          this.me.position[xyz] -= int;
        } // 如果撞牆就取消移動

        var xz = {x: this.me.position.x, z: this.me.position.z }
        
        this.camera.position.set(xz.x, this.me.position.y + 20, xz.z);
        this.controls.target.set(this.me.position.x, this.me.position.y, this.me.position.z);
        this.controls.update();
      }

      if (this.me.position.x === this.you.position.x && this.me.position.z === this.you.position.z ) {
        this.victory()
      }

      this.animateThreeJs();
    }
  }
}
</script>

<style type="text/css" scoped="">

#lev {
  position: fixed;
  top: 1em;
  right: 1em;
  color: white;
  background-color: black;
  padding: 3px;
}

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
  width: 10vw;
  height: 10vh;
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