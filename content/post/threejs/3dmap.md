---
layout: post
title: "ThreeJS实现3D中国地图"
subtitle: ""
date: 2023-02-27 17:58:34
author: "汪洋龙"
categories: [threejs]
image: "img/coffee-bg.jpg"
description: "告别伪3D地图，升级视觉体验"
tags:
  - "react"
  - "threejs"
---

# 实现 3D 中国地图

## 1. 示例

![3dmap](/post/threejs/images/3dmap.gif)

## 2. 构建基础模板

### 2.1 安装依赖项

```bash
npm i three d3 axios
npm i @types/three @types/d3 -D
```

```json
{
  "name": "demo-threejs-chinamap",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    ...
    "axios": "^1.3.4",
    "d3": "^7.8.2",
    "three": "^0.150.1",
  },
  ...
  "devDependencies": {
    "@types/d3": "^7.4.0",
    "@types/three": "^0.149.0"
  }
}
```

### 2.2 构建基础模板

```tsx
import { useCallback, useEffect, useRef, useState } from "react";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

function Map3D() {
  const mapRef = useRef<any>();

  useEffect(() => {
    const currentDom = mapRef.current;

    /**
     * 初始化场景
     */
    const scene = new THREE.Scene();

    /**
     * 初始化摄像机
     */
    const camera = new THREE.PerspectiveCamera(
      30,
      currentDom.clientWidth / currentDom.clientHeight,
      0.1,
      1000
    );
    camera.position.set(-10, -90, 130);

    /**
     * 初始化渲染器
     */
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(currentDom.clientWidth, currentDom.clientHeight);
    // 防止开发时重复渲染
    if (!currentDom.hasChildNodes()) {
      currentDom.appendChild(renderer.domElement);
    }

    /**
     * 初始化模型（地图模型绘制的逻辑将在这里替换）
     */
    const geometry = new THREE.BoxGeometry(1, 2, 3);
    const material = new THREE.MeshBasicMaterial({ color: "#fff" });
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    /**
     * 初始化 CameraHelper
     */
    const helper = new THREE.CameraHelper(camera);
    scene.add(helper);

    /**
     * 初始化 AxesHelper
     */
    const axesHelper = new THREE.AxesHelper(100);
    scene.add(axesHelper);

    /**
     * 初始化控制器
     */
    new OrbitControls(camera, renderer.domElement);

    const animate = function () {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    };
    animate();
  }, []);

  return <div ref={mapRef} style={{ width: "100%", height: "100%" }}></div>;
}

export default Map3D;
```

记得将所在的 DOM 元素撑开，这里在 `index.css` 中添加下面的代码，否则会报错 `THREE.BufferGeometry.computeBoundingSphere(): Computed radius is NaN. The "position" attribute is likely to have NaN values.`

```css
body
/* ... */
code
/* ... */

html,
body {
  height: 100%;
}

#root {
  height: 100%;
}
```

![step1](/post/threejs/images/step1.gif)

### 2.3 解决 onresize 问题

```tsx
...
useEffect(() => {
  ...
  // 防止开发时重复渲染
  // if (!currentDom.hasChildNodes()) {
  //   currentDom.appendChild(renderer.domElement);
  // }
  // 这里修改为下面写法，否则 onresize 不生效
  if (currentDom.childNodes[0]) {
    currentDom.removeChild(currentDom.childNodes[0]);
  }
  currentDom.appendChild(renderer.domElement);
  ...

  window.addEventListener("resize", onResize, false);
  return () => {
    window.removeEventListener("resize", onResize);
  };
}, [])
...
```

![step2](/post/threejs/images/step2.gif)

## 3. 绘制地图模型

通过 [GeoJson](https://geojson.org/) 生成地图模型，这里通过 [datav 地理小工具](http://datav.aliyun.com/portal/school/atlas/area_selector) 在线获取地图数据

### 3.1 `GeoJson` 数据格式

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "adcode": 110000,
        "name": "北京市",
        "center": [116.405285, 39.904989],
        "centroid": [116.41995, 40.18994],
        "childrenNum": 16,
        "level": "province",
        "parent": { "adcode": 100000 },
        "subFeatureIndex": 0,
        "acroutes": [100000]
      },
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [[[[]]]]
      }
    }
  ]
}
```

- `adcode`: 地图下钻的时候会根据 `adcode` 请求不同的数据
- `center`: 表示行政中心
- `centroid`: 表示形心
- `level`: 地图下钻的时候可以根据 `level` 实现对应视图的放大
- `geometry` 的 `coordinates`: 绘制地图的关键数据，会根据 `geometry` 的 `type` 表现出不同的多维数组

![geojsonbasic](/post/threejs/images/geojsonbasic.jpg)
![geojsonmulti](/post/threejs/images/geojsonmulti.jpg)

地图数据可能是 `MultiPolygon` 或者 `Polygon` 类型，所以它们是一个 `4` 维或者 `3` 维数组

### 3.2 墨卡托投影转换

`GeoJson` 的坐标需要进行 `墨卡托投影` 转换才能转换成平面坐标，这里需要用到 `d3`

```tsx
// typed.ts
import * as d3 from "d3";

// 返回的是一个函数
const projectionFn = d3
  .geoMercator()
  .center([104.0, 37.5]) // 中国中心点坐标，
  .scale(80) // 放大系数
  .translate([0, 0]);
```

### 3.3 解析 `GeoJSON`，通过 `new THREE.ExtrudeGeometry` 生成地图模型

定义数据类型

```ts
import { Object3D } from "three";

export interface GeoJsonType {
  type: "FeatureCollection";
  features: GeoJsonFeature[];
}

export interface GeoJsonFeature {
  type: string; // "Feature"
  properties: {
    adcode: number; //110000
    name: string; // 北京
    center: [number, number]; //[116.405285, 39.904989],
    centroid: [number, number]; //[116.419889, 40.189911]
    childrenNum: number; //16,
    level: Geolevel; // province,
    parent: {
      adcode: number; //100000
    };
    subFeatureIndex: number; //0,
    acroutes: number[]; // [100000],
    adchar: null;
  };
  geometry: {
    type: GeometryType; // "MultiPolygon",
    coordinates: GeometryCoordinates<GeometryType>;
  };
  vector3: any[][]; // 每个省份一个维度
}

export type Geolevel = "province" | "city" | "district";

export type GeometryType =
  | "Point"
  | "LineString"
  | "Polygon"
  | "MultiPoint"
  | "MultiLineString"
  | "MultiPolygon"
  | "GeometryCollection";

export type GeometryCoordinates<T extends GeometryType> = T extends "Point"
  ? [number, number]
  : T extends "LineString"
  ? [number, number][]
  : T extends "Polygon"
  ? [number, number][][]
  : T extends "MultiPoint"
  ? [number, number][]
  : T extends "MultiLineString"
  ? [number, number][][]
  : T extends "MultiPolygon"
  ? [number, number][][][]
  : T extends "GeometryCollection"
  ? any
  : never;

export interface ExtendObject3D extends Object3D {
  customProperties: any; // 扩展的自定义属性
}
```

绘制模型，这里用了 `Shader` 表示侧边材质的渐变

```tsx
// 绘制挤出的材质
export function drawExtrudeMesh(
  point: [number, number][],
  projectionFn: any
): any {
  const shape = new THREE.Shape();

  for (let i = 0; i < point.length; i++) {
    const [x, y]: any = projectionFn(point[i]); // 将每一个经纬度转化为坐标点
    if (i === 0) {
      shape.moveTo(x, -y);
    }
    shape.lineTo(x, -y);
  }

  const geometry = new THREE.ExtrudeGeometry(shape, {
    depth: 2, // 挤出的形状深度
    bevelEnabled: false, // 对挤出的形状应用是否斜角
  });

  const material = new THREE.MeshBasicMaterial({
    color: "#06092A",
    transparent: true,
    opacity: 0.9,
  });

  const materialSide = new THREE.ShaderMaterial({
    uniforms: {
      color1: {
        value: new THREE.Color("#3F9FF3"),
      },
      color2: {
        value: new THREE.Color("#266BF0"),
      },
    },
    vertexShader: `
        varying vec2 vUv;
        void main() {
          vUv = uv;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);
        }
      `,
    fragmentShader: `
          uniform vec3 color1;
          uniform vec3 color2;
          varying vec2 vUv;
          void main() {
            gl_FragColor = vec4(mix(color1, color2, vUv.y), 1.0);
          }
        `,
    //   wireframe: true,
  });

  const mesh = new THREE.Mesh(geometry, [material, materialSide]);
  return { mesh };
}

// 生成地图3D模型
export function generateMapObject3D(
  mapdata: GeoJsonType,
  center: [number, number] = [104.0, 37.5]
) {
  // 地图对象
  const mapObject3D = new THREE.Object3D();
  // 地图数据
  const { features: basicFeatures } = mapdata;

  const projectionFn = d3
    .geoMercator()
    .center(center)
    .scale(80)
    .translate([0, 0]);

  // 每个省的数据
  basicFeatures.forEach((basicFeatureItem: GeoJsonFeature) => {
    // 每个省份的地图对象
    const provinceMapObject3D = new THREE.Object3D() as ExtendObject3D;
    // 将地图数据挂在到模型数据上
    provinceMapObject3D.customProperties = basicFeatureItem.properties;

    // 每个坐标类型
    const featureType = basicFeatureItem.geometry.type;
    // 每个坐标数组
    const featureCoords: GeometryCoordinates<GeometryType> =
      basicFeatureItem.geometry.coordinates;
    // 每个中心点位置
    const featureCenterCoord: [number, number] =
      basicFeatureItem.properties.center;
    // 名字
    const featureName: string = basicFeatureItem.properties.name;

    // MultiPolygon 类型
    if (featureType === "MultiPolygon") {
      featureCoords.forEach((multiPolygon: [number, number][][]) => {
        multiPolygon.forEach((polygon: [number, number][]) => {
          const { mesh } = drawExtrudeMesh(polygon, projectionFn);
          provinceMapObject3D.add(mesh);
        });
      });
    }

    // Polygon 类型
    if (featureType === "Polygon") {
      featureCoords.forEach((polygon: [number, number][]) => {
        const { mesh } = drawExtrudeMesh(polygon, projectionFn);
        provinceMapObject3D.add(mesh);
      });
    }

    mapObject3D.add(provinceMapObject3D);
  });

  return mapObject3D;
}
```

![step3](/post/threejs/images/step3.gif)

### 3.4 解析 `GeoJSON`，绘制外线和内线

```tsx
// 绘制挤出的材质
function drawExtrudeMesh(point: [number, number][], projectionFn: any): any {
  const shape = new THREE.Shape();

  const vArr: any = [];
  const vlineArr: any = [];

  for (let i = 0; i < point.length; i++) {
    const [x, y]: any = projectionFn(point[i]); // 将每一个经纬度转化为坐标点
    if (!isNaN(x) || !isNaN(y)) {
      vArr.push(new THREE.Vector3(x, -y, 2));
      vlineArr.push(new THREE.Vector3(x, -y, 3));

      if (i === 0) {
        shape.moveTo(x, -y);
      }
      shape.lineTo(x, -y);
    }
  }

  const lineMaterial = new THREE.LineBasicMaterial({
    color: "#2977E7",
    linewidth: 1000 // 会发现设置宽度无效
  });
  const lineGeometry = new THREE.BufferGeometry();
  lineGeometry.setFromPoints(vArr);

  ...

  const mesh = new THREE.Mesh(geometry, [material, materialSide]);
  const line = new THREE.Line(lineGeometry, lineMaterial);

  return { mesh, line };
}
```

**Q1 `LineBasicMaterial` 设置 `linewidth` 无效的解决方案**

```tsx
const lineMaterial = new THREE.LineBasicMaterial({
  // color: "#fff",
  linewidth: 600,
  // dashSize: 0.3,
  // gapSize: 0.6,
});
```

原因：由于 OpenGL Core Profile 与 大多数平台上 WebGL 渲染器的限制，无论如何设置该值，线宽始终为 1。

解决方案：`Line2`

```tsx
import { Line2 } from "three/examples/jsm/lines/Line2";
import { LineGeometry } from "three/examples/jsm/lines/LineGeometry";
import { LineMaterial } from "three/examples/jsm/lines/LineMaterial";
```

**Q2 `THREE.BufferGeometry.computeBoundingSphere(): Computed radius is NaN. The "position" attribute is likely to have NaN values.`**

解析 `x`, `y` 坐标的时候会出现 `NaN`，需要处理判断下

```tsx
for (let i = 0; i < point.length; i++) {
  const [x, y]: any = projectionFn(point[i]); // 将每一个经纬度转化为坐标点
  if (!isNaN(x) || !isNaN(y)) {
    if (i === 0) {
      shape.moveTo(x, -y);
    }
    shape.lineTo(x, -y);
  }
}
```

## 4. 鼠标移入移出修改板块颜色

`webGL` 中获取鼠标交互物体的原理：通过三维空间中 `相机视点` 和 `鼠标在屏幕上的位置` 的连线，形成一条直线，捕获与此直线相交的空间中的物体，即为交互对象物体。在 `ThreeJS` 中通过 `Raycaster` 实现。

```tsx
/**
 * 设置 raycaster
 */
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();

// 鼠标移入事件
const onMouseMoveEvent = (e: MouseEvent) => {
  const intersects = raycaster.intersectObjects(scene.children);
  pointer.x = (e.clientX / currentDom.clientWidth) * 2 - 1;
  pointer.y = -(e.clientY / currentDom.clientHeight) * 2 + 1;

  // 如果存在，则鼠标移出需要重置
  if (lastPick) {
    lastPick.object.material[0].color.set("#06092A");
  }
  lastPick = null;
  lastPick = intersects.find(
    (item: any) => item.object.material && item.object.material.length === 2
  ); // 因为每个板块的材质是由2种材质组成，所以这里可以通过 length === 2 来判断

  if (lastPick) {
    if (lastPick.object.material[0]) {
      lastPick.object.material[0].color.set("#3497F5");
    }
  }
};

...
window.addEventListener("mousemove", onMouseMoveEvent, false);
...
window.removeEventListener("mousemove", onMouseMoveEvent);
...
```

![step4](/post/threejs/images/step4.gif)

**优化**：你会发现用 `item.object.material.length === 2` 去判断并不具备普适性，具体的业务场景会很复杂并且会构建很多 `Mesh`，甚至不够优雅。这里我们可以用 `Object3D` 官方的自定义属性 [userData](https://threejs.org/docs/#api/en/core/Object3D.userData) 进行改造。

```ts
...
export function drawExtrudeMesh() {
  ...
  const mesh: any = new THREE.Mesh(geometry, [material, materialSide]);
  // userData 存储自定义属性
  mesh.userData = {
    isChangeColor: true,
  };
  ...
}
...
```

```tsx
// lastPick = intersects.find(
//   (item: any) => item.object.material && item.object.material.length === 2
// );
// 优化
lastPick = intersects.find((item: any) => item.object.userData.isChangeColor);
```

## 5. 绘制 2d Tooltip

实现一个鼠标移动上去展示省份提示框信息的需求，这里绘制了一个固定的 `div` 用来展示，用 `visibility` 来控制显示和隐藏。

```tsx
...

  if (lastPick) {
    const properties = lastPick.object.parent.customProperties;
    if (lastPick.object.material[0]) {
      lastPick.object.material[0].color.set("#3497F5");
    }

    if (toolTipRef.current && toolTipRef.current.style) {
      toolTipRef.current.style.left = e.clientX + 2 + "px";
      toolTipRef.current.style.top = e.clientY + 2 + "px";
      toolTipRef.current.style.visibility = "visible";
    }
    setToolTipData({
      text: properties.name,
    });
  } else {
    toolTipRef.current.style.visibility = "hidden";
  }

...
return (
  <div
    style={{
      width: "100%",
      height: "100%",
      overflow: "hidden",
      position: "relative",
    }}
  >
    <div ref={mapRef} style={{ width: "100%", height: "100%" }}></div>
    <ToolTip innterRef={toolTipRef} data={toolTipData}></ToolTip>
  </div>
);
...
```

![step5](/post/threejs/images/step5.gif)

## 6. 绘制 2d Renderer

在地图中还需要有固定的省份信息展示，同时还要求与 3D 视角同步。这里用 `CSS2DRenderer` 绘制。

### 6.1 引入 CSS2DRenderer 包

```tsx
import {
  CSS2DRenderer,
  CSS2DObject,
} from "three/examples/jsm/renderers/CSS2DRenderer"; // 引入包
```

### 6.2 创建 css2 Renderer 渲染器

```tsx
...

/**
 * 创建css2 Renderer 渲染器
 */
const labelRenderer = new CSS2DRenderer();
labelRenderer.setSize(currentDom.clientWidth, currentDom.clientHeight);
labelRenderer.domElement.style.position = "absolute";
labelRenderer.domElement.style.top = "0px";
const labelRendererDom = map2dRef.current;
if (labelRendererDom?.childNodes[0]) {
  labelRendererDom.removeChild(labelRendererDom.childNodes[0]);
}
labelRendererDom.appendChild(labelRenderer.domElement);
```

### 6.3 通过数据绘制

```tsx
/**
 * 初始化模型（绘制3D模型）
 */
const { mapObject3D, label2dData } = generateMapObject3D(geoJson);
scene.add(mapObject3D);

/**
 * 绘制 2D 面板
 */
const labelObject2D = new THREE.Object3D();
label2dData.forEach((item: any) => {
  const { featureCenterCoord, featureName } = item;
  const labelObjectItem = draw2dLabel(featureCenterCoord, featureName);
  if (labelObjectItem) {
    labelObject2D.add(labelObjectItem);
  }
});
scene.add(labelObject2D);
```

### 6.4 修改轨道控制器

**在使用轨道控制器 OrbitControls 时，需要将原来绑定在 WebGLRenderer 的 DOM 改为绑定在 `CSS2DRenderer` 的 DOM 上，否则轨道控制器无法正常使用**

```tsx
/**
 * 初始化控制器
 */
// new OrbitControls(camera, renderer.domElement);
new OrbitControls(camera, labelRenderer.domElement);
```

### 6.5 渲染

```tsx
const animate = function () {
  ...
  labelRenderer.render(scene, camera);
};
```

### 6.6 适配 onresize

```tsx
// 视窗伸缩
function onResize() {
  ...
  labelRenderer.setSize(currentDom.clientWidth, currentDom.clientHeight);
}
```

![step6](/post/threejs/images/step6.gif)

## 7. 地图下钻

地图的下钻无非是数据的不同。这里通过 `dblclick` 事件获取对应的 `adcode` 请求不同的接口数据。有几个小点需要注意：

- 保证地图在页面中心需要传入 `center` 更新对应的 `projectionFn`
- 保证地图有合适的缩放比例需要传入 `scale` 更新对应的 `projectionFn`

![step7](/post/threejs/images/step7.gif)

## 8. 地图下钻动画

上面的地图下钻未免太过生硬，加入动画效果会更好。这里用 [gsap 动画库](https://greensock.com/docs/v3/GSAP) 实现。

### 8.1 安装依赖

```bash
npm i gsap
```

### 8.2 添加动画

```tsx
/**
 * 动画
 */
gsap.to(mapObject3D.scale, { x: 2, y: 2, z: 1, duration: 1 });
gsap.to(labelObject2D.scale, { x: 2, y: 2, z: 1, duration: 1 });
```

![step8](/post/threejs/images/step8.gif)

## 9. 绘制扩散点

地图上有扩散的点是一个很常见的需求，可以标注一些关键点位。

### 9.1 绘制

```ts
// 绘制圆点
export const drawSpot = (coord: [number, number]) => {
  if (coord && coord.length) {
    const POSITION_Z = 2.1;
    /**
     * 绘制圆点
     */
    const spotGeometry = new THREE.CircleGeometry(0.5, 200);
    const spotMaterial = new THREE.MeshBasicMaterial({
      color: "#3EC5FB",
      side: THREE.DoubleSide,
    });
    const circle = new THREE.Mesh(spotGeometry, spotMaterial);
    circle.position.set(coord[0], -coord[1], POSITION_Z);

    // 圆环
    const ringGeometry = new THREE.RingGeometry(0.5, 0.7, 50);
    const ringMaterial = new THREE.MeshBasicMaterial({
      color: "#3FC5FB",
      side: THREE.DoubleSide,
      transparent: true,
    });
    const ring = new THREE.Mesh(ringGeometry, ringMaterial);
    ring.position.set(coord[0], -coord[1], POSITION_Z);
    return { circle, ring };
  }
};
```

### 9.2 在 animate 函数中添加扩散逻辑

```tsx
const animate = function () {
  ...

  // 圆环
  spotList.forEach((mesh: any) => {
    mesh._s += 0.01;
    mesh.scale.set(1 * mesh._s, 1 * mesh._s, 1 * mesh._s);
    if (mesh._s <= 2) {
      mesh.material.opacity = 2 - mesh._s;
    } else {
      mesh._s = 1;
    }
  });
};
```

![step9](/post/threejs/images/step9.gif)

## 10. 扩散点之间的轨迹线

```ts
/**
 * 线上移动物体
 */
export const drawflySpot = (curve: any) => {
  const aGeo = new THREE.SphereGeometry(0.2);
  const aMater = new THREE.MeshBasicMaterial({
    color: "#77f077",
    side: THREE.DoubleSide,
  });
  const aMesh: any = new THREE.Mesh(aGeo, aMater);
  // 保存曲线实例
  aMesh.curve = curve;
  aMesh._s = 0;
  return aMesh;
};

// 绘制两点链接飞线
export const drawLineBetween2Spot = (
  coordStart: [number, number],
  coordEnd: [number, number]
) => {
  const [x0, y0, z0] = [...coordStart, POSITION_Z];
  const [x1, y1, z1] = [...coordEnd, POSITION_Z];
  // 使用 QuadraticBezierCurve3 创建 三维二次贝塞尔曲线
  const curve = new THREE.QuadraticBezierCurve3(
    new THREE.Vector3(x0, -y0, z0),
    new THREE.Vector3((x0 + x1) / 2, -(y0 + y1) / 2, 20),
    new THREE.Vector3(x1, -y1, z1)
  );

  const flySpot = drawflySpot(curve);

  const lineGeometry = new THREE.BufferGeometry();
  // 获取曲线上50个点
  const points = curve.getPoints(50);
  const positions = [];
  const colors = [];
  const color = new THREE.Color();

  // 给每个顶点设置演示 实现渐变
  for (let j = 0; j < points.length; j++) {
    color.setHSL(0.21 + j, 0.77, 0.55 + j * 0.0025); // 色
    colors.push(color.r, color.g, color.b);
    positions.push(points[j].x, points[j].y, points[j].z);
  }
  // 放入顶点 和 设置顶点颜色
  lineGeometry.setAttribute(
    "position",
    new THREE.BufferAttribute(new Float32Array(positions), 3, true)
  );
  lineGeometry.setAttribute(
    "color",
    new THREE.BufferAttribute(new Float32Array(colors), 3, true)
  );

  const material = new THREE.LineBasicMaterial({
    vertexColors: true,
    // color: "red",
    side: THREE.DoubleSide,
  });
  const flyLine = new THREE.Line(lineGeometry, material);

  return { flyLine, flySpot };
};
```

记得别忘了

```tsx
gsap.to(flyObject3D.scale, { x: 2, y: 2, z: 1, duration: 1 });
```

![step10](/post/threejs/images/step10.gif)

## 11. 雷达扫描效果

如果在地图背后加上雷达的扫描效果，是不是会很炫酷。

![step11](/post/threejs/images/step11.gif)

## 12. 光照效果

## 性能优化

threejs 频繁绘制下钻会出现掉帧现象

## 完整代码

[GitHub demo-threejs-chinamap](https://github.com/chemistwang/demo-threejs-chinamap)
