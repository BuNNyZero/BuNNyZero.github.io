# BuNNyZero.github.io

<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Three.js Starter Template</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            overflow: hidden; /* ซ่อน scrollbar */
            background-color: #000;
        }
        #webgl-canvas {
            width: 100vw;
            height: 100vh;
            display: block;
        }
    </style>
    <!-- Import Map สำหรับจัดการ ES Modules ของ Three.js -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
            }
        }
    </script>
</head>
<body>

    <!-- Canvas สำหรับแสดงผล 3D -->
    <canvas id="webgl-canvas"></canvas>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        // 1. Scene Setup
        const canvas = document.querySelector('#webgl-canvas');
        const scene = new THREE.Scene();

        // 2. Camera Setup
        const camera = new THREE.PerspectiveCamera(
            75, 
            window.innerWidth / window.innerHeight, 
            0.1, 
            1000
        );
        camera.position.set(0, 1, 3);

        // 3. Renderer Setup
        const renderer = new THREE.WebGLRenderer({
            canvas: canvas,
            antialias: true // เปิดใช้งาน Smooth Edges (กันขอบหยัก)
        });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

        // 4. OrbitControls (ช่วยให้ลากเมาส์หมุนกล้อง / ซูมได้)
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;

        // 5. Lights Setup
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 1.5);
        directionalLight.position.set(5, 5, 5);
        scene.add(directionalLight);

        // 6. Mesh (Geometry + Material)
        const geometry = new THREE.BoxGeometry(1, 1, 1);
        const material = new THREE.MeshStandardMaterial({ 
            color: 0x6366f1,
            roughness: 0.3,
            metalness: 0.2
        });
        const cube = new THREE.Mesh(geometry, material);
        scene.add(cube);

        // เพิ่ม Grid Helper พื้นหลังเพื่อความสวยงาม
        const gridHelper = new THREE.GridHelper(10, 10);
        scene.add(gridHelper);

        // 7. Handle Window Resize (รองรับการปรับขนาดหน้าจอ)
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        });

        // 8. Animation Loop
        const animate = () => {
            requestAnimationFrame(animate);

            // หมุนกล่องตัวอย่าง
            cube.rotation.x += 0.005;
            cube.rotation.y += 0.01;

            // อัปเดตการควบคุมกล้อง
            controls.update();

            // Render Render Render!
            renderer.render(scene, camera);
        };

        animate();
    </script>
</body>
</html>
