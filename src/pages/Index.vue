<template>
  <div></div>
</template>

<script>
import * as THREE from "three";
import Stats from "three/examples/jsm/libs/stats.module";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { Water } from "three/examples/jsm/objects/Water";
import { TWEEN } from "three/examples/jsm/libs/tween.module.min.js";
import {
  CSS2DObject,
  CSS2DRenderer,
} from "three/examples/jsm/renderers/CSS2DRenderer";
import waterTextureNormal from "@/assets/images/Water/Water_002_NORM.jpg";
import * as dat from "dat.gui";
import skybox_right from "@/assets/images/skybox/FS002_Day_Cubemap_right.jpg";
import skybox_left from "@/assets/images/skybox/FS002_Day_Cubemap_left.jpg";
import skybox_up from "@/assets/images/skybox/FS002_Day_Cubemap_up.jpg";
import skybox_down from "@/assets/images/skybox/FS002_Day_Cubemap_down.jpg";
import skybox_back from "@/assets/images/skybox/FS002_Day_Cubemap_back.jpg";
import skybox_front from "@/assets/images/skybox/FS002_Day_Cubemap_front.jpg";

var scene;
var camera;
var webGLRenderer;
var labelRenderer;
var stats;
var raycaster;
var orbitControl;
var water;

export default {
  name: "Index",
  data() {
    return {
      sizes: {
        width: window.innerWidth,
        height: window.innerHeight,
      },
    };
  },
  created() {
    this.InitBase();
    this.InitThree();
    this.InitWater();
    this.InitRaycaster();
    this.InitUI();
    this.InitOrbitControl();
    this.InitModel();
    this.Render();
  },
  methods: {
    //初始化基础组件及信息
    InitBase() {
      //初始化场景
      scene = new THREE.Scene();
      //初始化相机
      camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        2000
      );
      //初始化渲染器
      webGLRenderer = new THREE.WebGLRenderer({ antialias: true });
      //初始化状态
      stats = new Stats();
      //初始化CSS2D渲染器
      labelRenderer = new CSS2DRenderer();
      //初始化射线
      raycaster = new THREE.Raycaster();
      //初始化轨道控制器
      orbitControl = new OrbitControls(camera, webGLRenderer.domElement);
      //初始化补间动画
    },
    //初始化Three
    InitThree() {
      const sizes = this.sizes;
      camera.position.set(0, 0, 20);
      camera.aspect = sizes.width / sizes.height;
      camera.updateProjectionMatrix();
      scene.add(camera);

      webGLRenderer.outputEncoding = THREE.sRGBEncoding;
      webGLRenderer.setSize(sizes.width, sizes.height);
      window.addEventListener("resize", () => {
        camera.aspect = sizes.width / sizes.height;
        camera.updateProjectionMatrix();
        webGLRenderer.setSize(sizes.width, sizes.height);
      });
      document.body.appendChild(webGLRenderer.domElement);

      stats.setMode(0);
      stats.domElement.style.position = "absolute";
      stats.domElement.style.left = "0px";
      stats.domElement.style.top = "0px";
      document.body.appendChild(stats.domElement);

      // 初始化环境光
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
      const dirLight = new THREE.DirectionalLight(0xffffff, 1);
      dirLight.color.setHSL(0.1, 1, 0.95);
      dirLight.position.set(-1, 1.75, 1);
      dirLight.position.multiplyScalar(30);
      scene.add(ambientLight);
      scene.add(dirLight);
      //初始化天空盒
      var urls = [
        skybox_right,
        skybox_left,
        skybox_up,
        skybox_down,
        skybox_back,
        skybox_front,
      ];
      const textureCube = new THREE.CubeTextureLoader().load(urls);
      scene.background = textureCube;
    },
    //初始化水组件
    InitWater() {
      const waterGeometry = new THREE.PlaneGeometry(10000, 10000);
      water = new Water(waterGeometry, {
        textureWidth: 1024,
        textureHeight: 1024,
        waterNormals: new THREE.TextureLoader().load(
          waterTextureNormal,
          function (texture) {
            texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
          }
        ),
        sunDirection: new THREE.Vector3(),
        sunColor: 0xffffff,
        waterColor: 0x0072ff,
        distortionScale: 4,
        fog: scene.fog !== undefined,
      });
      water.rotation.x = -Math.PI / 2;
      water.position.set(0, 0, -60);
      scene.add(water);
    },
    //初始化射线组件
    InitRaycaster() {
      const raycaster = new THREE.Raycaster();
      const mouse = new THREE.Vector2();
      //对页面进行鼠标点击事件绑定
      window.addEventListener("mouseup", mouseup);
      //添加点击方法
      function mouseup(e) {
        if (e.button == 2) {
          // 将鼠标位置归一化为设备坐标。x 和 y 方向的取值范围是 (-1 to +1)
          mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
          mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;
          // 通过摄像机和鼠标位置更新射线
          raycaster.setFromCamera(mouse, camera);
          // 计算物体和射线的焦点
          const intersects = raycaster.intersectObjects(scene.children);
          alert(intersects[0].object.name);
        }
      }
    },
    //初始化模型
    InitModel() {
      const loadManager = new THREE.LoadingManager();
      const loader = new GLTFLoader(loadManager);
      const dracoLoader = new DRACOLoader("../../public/draco");
      loader.setDRACOLoader(dracoLoader);
      loader.load("../models/house/scene.gltf", (gltf) => {
        const model = gltf.scene;
        model.scale.set(0.05, 0.05, 0.05);
        model.position.set(0, 2, 0);
        scene.add(model);
      });
      loader.load("../models/bird/scene.gltf", (gltf) => {
        const model = gltf.scene;
        model.scale.set(5, 5, 5);
        model.position.set(50, 50, -25);
        scene.add(model);
      });
      loader.load("../models/boat/scene.gltf", (gltf) => {
        const model = gltf.scene;
        model.scale.set(5, 5, 5);
        model.position.set(60, 3.5, 40);
        scene.add(model);
      });
    },
    //初始化UI组件
    InitUI() {
      //dat.gui的使用
      var gui = new dat.GUI();
      var control = new (function () {
        this.spot1 = function () {
          this.CameraTween(50, 50, 50, 13, 107, -39);
        };
        this.spot2 = function () {
          this.CameraTween(-94, 8, -62, -60, 10, -50);
        };
        this.spot3 = function () {
          this.CameraTween(105, 24, 73, 60, 10, 40);
        };
        this.cameraPosition = function () {
          alert(camera.position);
        };
      })();
      var group1 = gui.addFolder("景点");
      group1.add(control, "spot1").name("海盗旗帜");
      group1.add(control, "spot2").name("鲨鱼群");
      group1.add(control, "spot3").name("小船");
      gui.add(control, "cameraPosition").name("查询摄像机位置");

      labelRenderer = new CSS2DRenderer();
      labelRenderer.setSize(window.innerWidth, window.innerHeight);
      labelRenderer.domElement.style.position = "absolute";
      labelRenderer.domElement.style.top = "0px";
      labelRenderer.domElement.style.left = "0px";
      labelRenderer.domElement.style.pointerEvents = "none";
      document.body.appendChild(labelRenderer.domElement);

      //创建交互点以及交互效果
      window.addEventListener("dblclick", () => {
        console.log("dbclick");
        this.CameraTween(50, 50, 50, 0, 0, 0);
      });
      this.CreatePoints(
        "1",
        13,
        107,
        29,
        "象征着海盗的旗帜，海盗在这里居住么？",
        [50, 50, 50, 13, 107, -39]
      );
      this.CreatePoints(
        "2",
        -60,
        10,
        50,
        "成群且凶猛的鲨鱼，可千万不能去招惹它们！",
        [-94, 8, -62, -60, 10, -50]
      );
      this.CreatePoints(
        "3",
        60,
        10,
        -40,
        "一艘划桨小木船，它的主人在哪？",
        [105, 24, 73, 60, 10, 40]
      );
    },
    //初始化轨道控制器组件
    InitOrbitControl() {
      orbitControl = new OrbitControls(camera, webGLRenderer.domElement);
      orbitControl.enableDamping = true;
      orbitControl.enablePan = false;
      orbitControl.maxPolarAngle = 1.5;
      orbitControl.minDistance = 50;
      orbitControl.maxDistance = 1200;
    },
    //渲染函数（类似Update）
    Render() {
      requestAnimationFrame(this.Render.bind(this));
      //水动画控制
      water.material.uniforms["time"].value += 1 / 60;
      //轨道控制器动画控制
      orbitControl && orbitControl.update();
      //renderer持续渲染
      webGLRenderer.render(scene, camera);
      labelRenderer.render(scene, camera);
      //性能检测更新
      stats.update();
      //补间动画更新
      TWEEN.update();
    },
    //-------------------------------封装-----------------------------------
    //创建交互点
    CreatePoints(number, worldX, worldY, worldZ, container, target = null) {
      const pointDiv = document.createElement("div");
      pointDiv.id = "point-" + number;
      pointDiv.textContent = number;
      pointDiv.style.padding = "4px 10px";
      pointDiv.style.color = "#fff";
      pointDiv.style.fontSize = "16px";
      pointDiv.style.position = "absolute";
      pointDiv.style.backgroundColor = "rgba(25,25,25,0.5)";
      pointDiv.style.borderRadius = "5px";
      pointDiv.style.pointerEvents = "auto";

      const pointLabel = new CSS2DObject(pointDiv);
      pointLabel.position.set(worldX, worldY, -worldZ);

      pointLabel.element.addEventListener("click", (e) => {
        const containerDiv = document.getElementById("container");
        if (containerDiv != null) {
          containerDiv.remove();
        }
        if (target != null) {
          this.CameraTween(
            target[0],
            target[1],
            target[2],
            target[3],
            target[4],
            target[5]
          );
        }
      });
      pointLabel.element.addEventListener("mouseenter", (e) => {
        let transformStr = {
          left: parseFloat(this.StringOperation(e)[0]),
          top: parseFloat(this.StringOperation(e)[1]),
        };
        this.CreateContainer(container, transformStr);
      });
      pointLabel.element.addEventListener("mouseleave", (e) => {
        const containerDiv = document.getElementById("container");
        if (containerDiv != null) {
          containerDiv.remove();
        }
      });
      scene.add(pointLabel);
    },
    //摄像机动画控制
    CameraTween(posX, posY, posZ, targetX, targetY, targetZ) {
      var para = {
        x: camera.position.x,
        y: camera.position.y,
        z: camera.position.z,
        x1: orbitControl.target.x,
        y1: orbitControl.target.y,
        z1: orbitControl.target.z,
      };
      var tween = new TWEEN.Tween(para)
        .to(
          {
            x: posX,
            y: posY,
            z: posZ,
            x1: targetX,
            y1: targetY,
            z1: targetZ,
          },
          1000
        )
        .onStart(() => {
          orbitControl.enabled = false;
        })
        .onUpdate(() => {
          camera.position.set(para.x, para.y, para.z);
          orbitControl.target.set(para.x1, para.y1, para.z1);
        })
        .onComplete(() => {
          orbitControl.enabled = true;
        })
        .easing(TWEEN.Easing.Linear.None)
        .start();
    },
    //transform字符操作
    StringOperation(e) {
      const str1 = e.target.style.transform.toString().substring(31);
      const strs = str1.split(",");
      const leftStr = strs[0].slice(1, -2);
      const topStr = strs[1].slice(0, -3);
      return [leftStr, topStr];
    },
    //创建交互信息
    CreateContainer(text, transformStr) {
      const containerDiv = document.createElement("div");
      containerDiv.id = "container";
      containerDiv.style.position = "absolute";
      containerDiv.style.width = "80px";
      containerDiv.style.left = transformStr.left - 50 + "px";
      containerDiv.style.top = transformStr.top + 30 + "px";
      containerDiv.innerText = text;
      containerDiv.style.zIndex = 999;
      containerDiv.style.padding = "4px 10px";
      containerDiv.style.color = "#fff";
      containerDiv.style.fontSize = "16px";
      containerDiv.style.backgroundColor = "rgba(25,25,25,0.5)";
      containerDiv.style.borderRadius = "5px";
      containerDiv.style.pointerEvents = "none";
      document.body.appendChild(containerDiv);
    },
  },
};
</script>

<style></style>
