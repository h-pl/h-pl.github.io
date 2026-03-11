---
title: 'Three.js 基础实例'
date: 2022-01-11 18:54:30
tags: [Threejs,jsNote]
published: true
hideInList: true
feature: 
isTop: false
---
1. 基础知识
Threejs渲染其实和使用Lumion渲染十分类似，无论是渲染图片还是视频。一样的步骤，创建创景→相机拍摄→渲染出图或视频。让人十分幸福的是，场景（Scene）、相机（Camera）、渲染器（Renderer）这三种都可以十分方便的建立。**唯一不同的是**，Lumion渲染器是软件自带的，threejs要自己创建渲染器。（渲染器其实是对不同材质的解释器。）
    ```js
    //场景
    const scene = new THREE.Scene();
    //相机-四个参数（视角，相片宽高比，近截面，远截面）
    const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / 
    window.innerHeight, 0.1, 1000 );
    //渲染器
    const renderer = new THREE.WebGLRenderer();
    //渲染器大小
    renderer.setSize( window.innerWidth, window.innerHeight );
    document.body.appendChild( renderer.domElement );
    ```

   - 对Webgl**封装**，实现 诸如场景、灯光、阴影、材质、贴图、空间运算等功能
  ![](https://h-pl.github.io/post-images/1641899560701.png)
   - 这些对象通过一个层级关系明确的树状结构来展示出各自的位置和方向。子对象的位置和方向总是相对于父对象而言的。
   - **网格不是最小单位**，网格包含几何体和材质。
   - 相同几何体A、B、C为一个几何体A，其他网格内的相同几何体B、C以**引用**的方式访问A。 例如，在不同的位置画两个蓝色立方体，我们会需要两个网格(Mesh)对象来代表每一个立方体的位置和方向。但只需一个几何体(Geometry)来存放立方体的顶点数据，和一种材质(Material)来定义立方体的颜色为蓝色就可以了。两个网格(Mesh)对象都引用了相同的几何体(Geometry)和材质(Material)。
        - 几何体并不是一般意义的**几何体**，更多的是**顶点信息**。Three.js内置了许多[基本几何体](https://threejs.org/manual/#zh/primitives) 。你也可以创建自定义几何体或从文件中加载几何体。
        - 材质(Material)对象代表绘制几何体的表面属性，包括使用的颜色，和光亮程度。一个材质(Material)可以引用一个或多个纹理(Texture)，
    - 光源(Light)对象代表不同种类的光。
2. 从 hello word 开始

    2.1 在html内```<script>```中引入**three.js** 
    ```js
    <script type="module">
    import * as THREE from '../../build/three.module.js';
    </script>
    ```
    
    2.2 html内创建```<canvas>```标签，并注明id
    ```html
    <body>
    <canvas id="c"></canvas>
    </body>
    ```
    2.3 链接画布并挂载渲染器
    ```js
    <script type="module">
    import * as THREE from '../../build/three.module.js';
    
    function main() {
    const canvas = document.querySelector('#c'); //获取画布
    const renderer = new THREE.WebGLRenderer({canvas});//挂载渲染器
    ...
    </script>
    ```
    2.4 创建**透视视图**(近大远小)摄像机，只有视锥内的物体才能被看到
    ```js
    const fov = 75;
    const aspect = 2;  // 相机默认值
    const near = 0.1;
    const far = 5;
    const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
    //这四个参数定义了一个 "视椎(frustum)"。
    ```
    - 视野范围(field of view)的缩写。上述代码中是指垂直方向为75度。 注意three.js中大多数的角用弧度表示，但是因为某些原因透视摄像机使用角度表示。
    - aspect指画布的宽高比。我们将在别的文章详细讨论，在默认情况下 画布是300x150像素，所以宽高比为300/150或者说2。
    - near和far代表近平面和远平面，它们限制了摄像机面朝方向的可绘区域。 任何距离小于或超过这个范围的物体都将被裁剪掉(不绘制)。
    ![视椎](https://h-pl.github.io/post-images/1641902032047.png)
    - 近平面和远平面的高度由视野范围决定，宽度由视野范围和宽高比决定。

    2.5 创建一个场景(Scene)
    ```js
    const scene = new THREE.Scene();
    //欲创建一个mesh，则需创建几何体和材质

    //建模-创建几何体，一个包含盒子信息的立方几何体(BoxGeometry)
    const boxWidth = 1;
    const boxHeight = 1;
    const boxDepth = 1;
    const geometry = new THREE.BoxGeometry(boxWidth, boxHeight, boxDepth);

    //创建材质，然后创建一个基本的材质并设置它的颜色. 
    //颜色的值可以用css方式和十六进制来表示。
    const material = new THREE.MeshBasicMaterial({color: 0x44aa88});

    //创建网格
    const cube = new THREE.Mesh(geometry, material);

    //导入场景中
    scene.add(cube);

    //渲染出来
    renderer.render(scene, camera);
    ```




