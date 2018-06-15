<template>
<div></div>
</template>
<script>
/* eslint-disable */
const THREE = require('three')
const OrbitControls = require("three-orbit-controls")(THREE)
import { TweenMax } from 'gsap/TweenMax'

let raycaster
let mouse
let radius = Math.pow(100, 2)
let vx = 0
let vy = 0
export default {
  name: 'home',
  data() {
    return {
      camera: '', scene: '', renderer: '', geo: '', mat: '', fontLoader: '', objLoader: '', orbitRadius: 1, fontMesh: null, originalVertices: null
    };
  },
  methods: {
    onMouseMove: function(event) {
      	event.preventDefault();
				mouse.x = ( event.clientX / window.innerWidth ) * 2 - 1;
				mouse.y = - ( event.clientY / window.innerHeight ) * 2 + 1;
        // console.log(mouse)
    },
    init: async function() {
        this.fontLoader = new THREE.FontLoader();

        raycaster = new THREE.Raycaster();
        mouse = new THREE.Vector2(), THREE.INTERSECTED

        this.scene = new THREE.Scene();
        this.scene.background = new THREE.Color(0xffffff)

        this.mat = new THREE.MeshNormalMaterial();

        this.fontMesh = await this.loadFont()

        // this.scene.add(fontMesh);

        this.originalVertices = this.fontMesh.geometry.vertices
        this.originalVertices.forEach((vert, i) => {
            const geo = new THREE.BoxGeometry(0.2, 0.2, 0.2);
            const mesh = new THREE.Mesh(geo, this.mat);
            // const pivot = new THREE.Object3D()
            // pivot.add(mesh)
            // pivot.position.set(vert.x, vert.y, vert.z);
            // this.scene.add(pivot)
            mesh.position.set(vert.x, vert.y, vert.z);
            this.scene.add(mesh);
        })
        this.setRenderer();
        this.setCamera();
        this.setOrbitControls();
        document.addEventListener( 'mousemove', this.onMouseMove, false )
        this.animate();
    },
    randomFloat: function(min, max) {
      return Math.random() * (max - min) + min;
    },
    animate: function() {
        // this.camera.updateMatrixWorld()
        raycaster.setFromCamera( mouse, this.camera );
        var intersects = raycaster.intersectObjects( this.scene.children );
        if ( intersects.length > 0 ) {
          console.log('INTERSECTS!!', intersects)
          intersects.forEach((m) => {
            const pos = m.object.position
            pos.x = -this.randomFloat(-1, 1) * mouse.x
            pos.y = -this.randomFloat(-1, 1) * mouse.y
            TweenMax.to(pos, 5, { x: pos.x, y: pos.y })
          })
          // const mesh = intersects[0].object
          // mesh.position.x = 0
          // const rx = mouse.x
          // const ry = mouse.y
          // const distance = rx * rx + ry * ry
          // const force = -radius / distance
          // if(distance < radius) {
          //   const angle = Math.atan2(ry, rx)
          //   vx += force * Math.cos(angle)
          //   vy += force * Math.sin(angle)
          //   TweenMax.to(mesh.position, 5, { x: vx, y: vy })
            return this.render()
          // }

        }
        this.originalVertices.forEach((vert, i) => {
          const mesh = this.scene.children[i]
          if (TweenMax.isTweening(mesh.position)) return false
          const posX = vert.x
          const posY = vert.y
          const posZ = vert.z
          const x = this.randomFloat(posX -0.8, posX + 0.8)
          const y = this.randomFloat(posY -0.8, posY + 0.8)
          const z = this.randomFloat(posZ -0.8, posZ + 0.8)
          mesh.rotation.x += 0.01
          mesh.rotation.y += 0.01
          mesh.rotation.z += 0.01

          TweenMax.to(mesh.position, 5, { x, y, z, ease: Power0.easeNone })
        })
        return this.render()
    },
    render: function() {
        this.renderer.render(this.scene, this.camera);
        requestAnimationFrame(this.animate);
    },

    rotateAboutPoint: function(obj, point, axis, theta, pointIsWorld) {
      pointIsWorld = (pointIsWorld === undefined)? false : pointIsWorld;

      if(pointIsWorld){
        obj.parent.localToWorld(obj.position); // compensate for world coordinate
      }

      obj.position.sub(point); // remove the offset
      obj.position.applyAxisAngle(axis, theta); // rotate the POSITION
      obj.position.add(point); // re-add the offset

      if(pointIsWorld){
        obj.parent.worldToLocal(obj.position); // undo world coordinates compensation
      }

      obj.rotateOnAxis(axis, theta); // rotate the OBJECT
    },

    setOrbitControls: function() {
        new OrbitControls(this.camera, this.renderer.domElement);
    },

    setCamera: function() {
        this.camera = new THREE.PerspectiveCamera(
            90,
            window.innerWidth / window.innerHeight,
            0.1,
            50
        );
        this.camera.position.z = 30;
        this.camera.lookAt(0, 0, 0);
    },

    setRenderer: function() {
        this.renderer = new THREE.WebGLRenderer({ antialias: true });
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(this.renderer.domElement);
    },

    loadFont: function() {
        return new Promise((resolve, reject) =>
            this.fontLoader.load(
                "./static/Roboto.json",
                font => {
                    this.geo = new THREE.TextGeometry('s', {
                        font,
                        size: 20,
                        height: 0.1,
                        curveSegments: 4,
                        bevelEnabled: true,
                        bevelThickness: 0.8,
                        bevelSize: 0,
                        bevelSegments: 3
                    });
                    resolve(new THREE.Mesh(this.geo, this.mat));
                },
                null,
                reject
            )
        );
    },
  },
  beforeMount() {
    this.init()
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
