<template>
<div></div>
</template>
<script>
/* eslint-disable */
const THREE = require('three')
const OrbitControls = require("three-orbit-controls")(THREE);

export default {
  name: 'home',
  data() {
    return {
      camera: '', scene: '', renderer: '', geo: '', mat: '', fontLoader: '', objLoader: '', orbitRadius: 1, fontMesh: null
    };
  },
  methods: {
    init: async function() {
        this.fontLoader = new THREE.FontLoader();

        this.setRenderer();
        this.setCamera();
        this.setOrbitControls();

        this.scene = new THREE.Scene();

        this.mat = new THREE.MeshNormalMaterial();

        this.fontMesh = await this.loadFont()

        // this.scene.add(fontMesh);

        this.fontMesh.geometry.vertices.forEach((vert, i) => {
            const geo = new THREE.BoxGeometry(0.1, 0.1, 0.1);
            const mesh = new THREE.Mesh(geo, this.mat);

            mesh.position.set(vert.x, vert.y, vert.z);

            this.scene.add(mesh);
        })

        this.animate();
    },
    animate: function() {
        requestAnimationFrame(this.animate);
        this.fontMesh.geometry.vertices.forEach((vert, i) => {
          const mesh = this.scene.children[i]
          // mesh.rotation.x += 0.1
          // mesh.position.x = Math.cos(vert.x * date % i)
          const date = new Date()
          const vert1 = vert
          vert1.y = Math.sin(0.001 * date)
          vert1.x = Math.cos(0.001 * date)
          mesh.rotation.x += 0.01
          this.rotateAboutPoint(mesh, vert1, vert, THREE.Math.degToRad(0.1))
          // mesh.position.y = Math.sin(date) * vert.y
        })
        this.renderer.render(this.scene, this.camera);
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
                    this.geo = new THREE.TextGeometry("e", {
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
