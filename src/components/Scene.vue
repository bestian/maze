<template>
  <div class="scene">
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
      controls: null
    }
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
    this.camera.position.set(15, 5, 15)
    
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
    var BoxGeometry = new THREE.BoxGeometry(5, 1, 5)
    var maze = [
      [1,1,1,1,1,1,1,1,1,1,1,1,1,9,1],
      [1,0,0,0,0,0,0,0,1,0,0,0,0,0,1],
      [1,0,1,1,1,0,1,1,1,0,1,1,1,1,1],
      [1,0,0,0,1,1,1,0,1,0,1,1,1,0,1],
      [1,1,1,0,0,0,0,0,1,0,0,0,0,0,1],
      [1,0,0,0,1,1,0,1,1,0,1,1,1,0,1],
      [1,1,1,0,1,0,0,0,0,0,0,0,1,0,1],
      [1,0,0,0,1,1,0,1,1,0,1,1,1,0,1],
      [1,1,1,9,1,1,1,1,1,1,1,1,1,1,1]
    ];
    var meshArray = [];
    for (var x=0; x<maze.length; x++) {
        for(var z=0; z<maze[0].length; z++){
          var mesh;
          if (maze[x][z] === 1) {
            mesh = new THREE.Mesh(BoxGeometry, material.clone());
            meshArray.push(mesh);
            mesh.position.x = -5*x - 10;
            mesh.position.y = -20;
            mesh.position.z = -5*z + 20;
            this.scene.add(mesh);  
            console.log(meshArray.length, mesh.position);
          }
          if (maze[x][z] === 9) {
            var Spheregeometry = new THREE.SphereGeometry( 1, 32, 32 );
            var material2 = new THREE.MeshBasicMaterial( {color: 0xffff00} );
            mesh = new THREE.Mesh( Spheregeometry, material2.clone() );
            meshArray.push(mesh);
            mesh.position.x = -5*x - 10;
            mesh.position.y = -20;
            mesh.position.z = -5*z + 20;
            this.scene.add(mesh);  
            console.log(meshArray.length, mesh.position);
          }
        }
    }

    this.animateThreeJs()
  },
  methods: {
    animateThreeJs () {
      this.renderer.render(this.scene, this.camera)
      this.renderer.shadowMap.needsUpdate = true
    }
  }
}
</script>
