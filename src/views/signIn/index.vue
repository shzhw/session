<template>
  <div class="sign-in-wrap">
    <div id="container"></div>
    <div id="info">3D签到墙（使用官方demo源元素周期表修改）</div>
    <div id="menu">
      <button id="table" @click="transform(targets.table, 1000)">表格</button>
      <button id="sphere" @click="transform(targets.sphere, 2000)">球球</button>
      <button id="helix" @click="transform(targets.helix, 2000)">螺旋</button>
      <button id="grid" @click="transform(targets.grid, 2000)">格子</button>
    </div>

    <div class="show_info animated" style="display:none;">
      <div class="info_my">
        <img src="./img/c.png" />
        <div class="info_mem">
          <div class="nickname">smile的微笑</div>
          <div class="id">ID:123</div>
          <div class="vote">票数：123</div>
        </div>
      </div>
      <div class="intro">我想说的那些事你想听吗？曾经，在一个很远很远的地方，那里绿水长青，没有现在都市的喧哗，犹如室外桃园。。。想知道更多精彩内容吗?</div>
    </div>
  </div>
</template>

<script>
  /* eslint-disable no-redeclare */
  import $ from 'jquery';
  import THREE from './TrackballControls';
  import TWEEN from '@tweenjs/tween.js';

  import BIMG from './img/b.png';
  import AIMG from './img/a.png';

  export default {
    data() {
      return {
        personArray: [],
        targets: {
          table: [],
          sphere: [],
          helix: [],
          grid: []
        },
        camera: null,
        scene: null,
        renderer: null,
        controls: null,
        animateCode: 0,
        css3DObjects: []
      };
    },
    computed: {
      wallTable() {
        return this.personArray.map((item, index) => {
          return Object.assign({}, item, {
            src: item.image,
            p_x: (index % 20) + 1,
            p_y: Math.floor(index / 20) + 1
          })
        });
      }
    },
    mounted() {      
      for (var i = 0; i < 10; i++) {
        this.personArray.push({
          image: AIMG
        });
      }
      this.init();
      this.animate();
    },
    methods: {
      init() {
        this.camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 1, 10000);
        this.camera.position.z = 3000;

        this.scene = new THREE.Scene();

        this.createTableData();
        this.createSphereData();
        this.createHelixData();
        this.createGridData();

        //渲染
        this.renderer = new THREE.CSS3DRenderer();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        this.renderer.domElement.style.position = 'absolute';
        document.getElementById('container').appendChild(this.renderer.domElement);

        // 鼠标控制
        this.controls = new THREE.TrackballControls(this.camera, this.renderer.domElement);
        this.controls.rotateSpeed = 0.5;
        this.controls.minDistance = 500;
        this.controls.maxDistance = 6000;
        this.controls.addEventListener('change', this.render);

        // 自动更换
        setInterval(function() {
          this.animateCode = this.animateCode >= 3 ? 0 : this.animateCode;
          ++this.animateCode;
          switch (this.animateCode) {
            case 1:
              this.transform(this.targets.sphere, 1000);
              break;
            case 2:
              this.transform(this.targets.helix, 1000);
              break;
            case 3:
              this.transform(this.targets.grid, 1000);
              break;
          }
        }, 8000);

        this.transform(this.targets.table, 2000);
      },
      createTableData() {
        for (let i = 0; i < this.wallTable.length; i++) {
          let element = document.createElement('div');
          element.className = 'element';
          element.style.backgroundColor = 'rgba(0, 127, 127,' + (Math.random() * 0.5 + 0.25) + ')';

          let img = document.createElement('img');
          img.src = this.wallTable[i].image;
          element.appendChild(img);

          let object3D = new THREE.CSS3DObject(element);
          object3D.position.x = Math.random() * 4000 - 2000;
          object3D.position.y = Math.random() * 4000 - 2000;
          object3D.position.z = Math.random() * 4000 - 2000;
          this.scene.add(object3D);
          this.css3DObjects.push(object3D);

          // 表格需要坐标进行排序的
          let object = new THREE.Object3D();
          object.position.x = this.wallTable[i].p_x * 140 - 1330;
          object.position.y = -(this.wallTable[i].p_y * 180) + 990;

          this.targets.table.push(object);
        }
      },
      createSphereData() {
        let sphereVector = new THREE.Vector3();
        let sphereSpherical = new THREE.Spherical();
        for (let i = 0, l = this.wallTable.length; i < l; i++) {
          let phi = Math.acos(-1 + (2 * i) / l);
          let theta = Math.sqrt(l * Math.PI) * phi;

          let object = new THREE.Object3D();
          sphereSpherical.set(800, phi, theta);
          object.position.setFromSpherical(sphereSpherical);
          sphereVector.copy(object.position).multiplyScalar(2);
          object.lookAt(sphereVector);
          this.targets.sphere.push(object);
        }
      },
      createHelixData() {
        let helixVector = new THREE.Vector3();
        let helixCylindrical = new THREE.Cylindrical();
        for (let i = 0, l = this.wallTable.length; i < l; i++) {
          let theta = i * 0.175 + Math.PI;
          let y = -(i * 5) + 450;

          let object = new THREE.Object3D();

          // 参数一 圈的大小 参数二 左右间距 参数三 上下间距
          helixCylindrical.set(900, theta, y);

          object.position.setFromCylindrical(helixCylindrical);

          helixVector.x = object.position.x * 2;
          helixVector.y = object.position.y;
          helixVector.z = object.position.z * 2;

          object.lookAt(helixVector);

          this.targets.helix.push(object);
        }
      },
      createGridData() {
        for (let i = 0; i < this.wallTable.length; i++) {
          let object = new THREE.Object3D();

          object.position.x = (i % 5) * 400 - 800; // 400 图片的左右间距  800 x轴中心店
          object.position.y = -(Math.floor(i / 5) % 5) * 300 + 500; // 500 y轴中心店
          object.position.z = Math.floor(i / 25) * 200 - 800; // 300调整 片间距 800z轴中心店

          this.targets.grid.push(object);
        }
      },
      transform(targets, duration) {
        TWEEN.removeAll();

        for (let i = 0; i < this.css3DObjects.length; i++) {
          let object = this.css3DObjects[i];
          let target = targets[i];

          new TWEEN.Tween(object.position)
            .to({ x: target.position.x, y: target.position.y, z: target.position.z }, Math.random() * duration + duration)
            .easing(TWEEN.Easing.Exponential.InOut)
            .start();

          new TWEEN.Tween(object.rotation)
            .to({ x: target.rotation.x, y: target.rotation.y, z: target.rotation.z }, Math.random() * duration + duration)
            .easing(TWEEN.Easing.Exponential.InOut)
            .start();
        }

        new TWEEN.Tween(this)
          .to({}, duration * 2)
          .onUpdate(this.render)
          .start();
      },
      animate() {
        // 让场景通过x轴或者y轴旋转  & z
        this.scene.rotation.y += 0.008;
        requestAnimationFrame(this.animate);
        TWEEN.update();
        this.controls.update();
        // 渲染循环
        this.render();
      },
      render() {
        this.renderer.render(this.scene, this.camera);
      },
      addItem() {

      }
    }
  };
  /* eslint-enable no-redeclare */
</script>

<style lang="scss" scoped>
  .sign-in-wrap {
    background-color: #000000;
    height: 100vh;
  }
  a {
    color: #ffffff;
  }

  #info {
    position: absolute;
    width: 100%;
    color: #ffffff;
    padding: 5px;
    font-family: Monospace;
    font-size: 13px;
    font-weight: bold;
    text-align: center;
    z-index: 1;
  }

  #menu {
    position: absolute;
    bottom: 20px;
    width: 100%;
    text-align: center;
  }

  .element {
    width: 100px;
    height: 100px;
    box-shadow: 0px 0px 12px rgba(0, 255, 255, 0.5);
    border: 1px solid rgba(127, 255, 255, 0.25);
    text-align: center;
    cursor: default;
  }

  .element:hover {
    box-shadow: 0px 0px 12px rgba(0, 255, 255, 0.75);
    border: 1px solid rgba(127, 255, 255, 0.75);
  }

  .element img {
    width: 100px;
    height: 100px;
  }

  .element .number {
    position: absolute;
    top: 20px;
    right: 20px;
    font-size: 12px;
    color: rgba(127, 255, 255, 0.75);
  }

  .element .symbol {
    position: absolute;
    top: 40px;
    left: 0px;
    right: 0px;
    font-size: 60px;
    font-weight: bold;
    color: rgba(255, 255, 255, 0.75);
    text-shadow: 0 0 10px rgba(0, 255, 255, 0.95);
  }

  .element .details {
    position: absolute;
    bottom: 15px;
    left: 0px;
    right: 0px;
    font-size: 12px;
    color: rgba(127, 255, 255, 0.75);
  }

  button {
    color: rgba(127, 255, 255, 0.75);
    background: transparent;
    outline: 1px solid rgba(127, 255, 255, 0.75);
    border: 0px;
    padding: 5px 10px;
    cursor: pointer;
  }

  button:hover {
    background-color: rgba(0, 255, 255, 0.5);
  }

  button:active {
    color: #000000;
    background-color: rgba(0, 255, 255, 0.75);
  }

  .show_info {
    position: fixed;
    background-color: rgba(0, 0, 0, 0.6);
    padding: 10px;
    width: 300px;
    margin: 0 auto;
    left: 0;
    right: 0;
    border-radius: 5px;
    box-shadow: 0 0 10px 0 #fff;
    top: 30%;
  }

  .show_info img {
    display: block;
    margin: auto;
    border-radius: 5px;
    box-shadow: 0 0 10px 0 #888;
  }

  .show_info .intro {
    color: #fff;
    text-indent: 20px;
    margin-top: 10px;
    height: 65px;
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
  }

  .show_info .info_my {
    text-align: center;
  }

  .show_info .info_my > * {
    display: inline-block !important;
    vertical-align: middle;
  }

  .show_info .info_my .info_mem {
    color: #fff;
    max-width: 120px;
  }

  .show_info .info_my .info_mem > div {
    text-align: left;
  }

  .show_info .info_my .info_mem > div.nickname {
    max-width: 120px;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }
</style>
