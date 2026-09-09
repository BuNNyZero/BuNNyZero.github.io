# BuNNyZero.github.io

<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Flying Sword - Three.js</title>
    <style>
        /* บังคับให้แสดงผลเต็มหน้าจอ 100% */
        html, body {
            width: 100%;
            height: 100%;
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #05050d;
            font-family: sans-serif;
        }

        #webgl-canvas {
            width: 100vw;
            height: 100vh;
            display: block;
            position: absolute;
            top: 0;
            left: 0;
        }

        /* ข้อความแนะนำวิธีเล่น */
        .instructions {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            color: rgba(255, 255, 255, 0.7);
            font-size: 14px;
            pointer-events: none;
            user-select: none;
            text-align: center;
            background: rgba(0, 0, 0, 0.5);
            padding: 8px 16px;
            border-radius: 20px;
            backdrop-filter: blur(4px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
    </style>
    <!-- Import Map สำหรับ Three.js -->
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

    <canvas id="webgl-canvas"></canvas>
    
    <div class="instructions">
        🖱️ คลิกซ้ายลากเพื่อหมุนกล้อง | คลิกขวาเพื่อเลื่อน | สโครลเพื่อซูม | คลิกที่กระบี่เพื่อปล่อยพลัง!
    </div>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        // 1. Scene Setup
        const canvas = document.querySelector('#webgl-canvas');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x05050d, 0.03); // หมอกเพิ่มบรรยากาศแฟนตาซี

        // 2. Camera Setup
        const camera = new THREE.PerspectiveCamera(
            60, 
            window.innerWidth / window.innerHeight, 
            0.1, 
            1000
        );
        camera.position.set(0, 3, 10);

        // 3. Renderer Setup
        const renderer = new THREE.WebGLRenderer({
            canvas: canvas,
            antialias: true
        });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.shadowMap.enabled = true;

        // 4. OrbitControls
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;

        // 5. Lighting Setup
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
        scene.add(ambientLight);

        const mainLight = new THREE.DirectionalLight(0x90e0ef, 2.5);
        mainLight.position.set(5, 10, 7);
        scene.add(mainLight);

        // แสงสว่างสีฟ้าปล่อยออกจากกระบี่ (Glow Light)
        const swordLight = new THREE.PointLight(0x00f0ff, 3, 8);
        scene.add(swordLight);

        // 6. ฟังก์ชันสร้างโมเดลกระบี่ 3D (Procedural Chinese Flying Sword)
        function createSword() {
            const swordGroup = new THREE.Group();

            // วัสดุ (Materials)
            const bladeMaterial = new THREE.MeshStandardMaterial({
                color: 0xe0e0e0,
                metalness: 0.95,
                roughness: 0.1,
            });

            const goldMaterial = new THREE.MeshStandardMaterial({
                color: 0xffd700,
                metalness: 0.8,
                roughness: 0.2,
            });

            const handleMaterial = new THREE.MeshStandardMaterial({
                color: 0x4a0e17,
                roughness: 0.6,
            });

            // 6.1 ตัวคมกระบี่ (Blade)
            const bladeShape = new THREE.Shape();
            bladeShape.moveTo(0, 0);
            bladeShape.lineTo(0.12, 0.1);
            bladeShape.lineTo(0.08, 3.5);
            bladeShape.lineTo(0, 4.0); // ปลายแหลม
            bladeShape.lineTo(-0.08, 3.5);
            bladeShape.lineTo(-0.12, 0.1);
            bladeShape.closePath();

            const extrudeSettings = {
                depth: 0.03,
                bevelEnabled: true,
                bevelSegments: 2,
                steps: 1,
                bevelSize: 0.02,
                bevelThickness: 0.02
            };

            const bladeGeo = new THREE.ExtrudeGeometry(bladeShape, extrudeSettings);
            bladeGeo.center();
            const blade = new THREE.Mesh(bladeGeo, bladeMaterial);
            blade.position.y = 2.0;
            swordGroup.add(blade);

            // 6.2 กระบังมือ (Guard)
            const guardGeo = new THREE.BoxGeometry(0.8, 0.15, 0.2);
            const guard = new THREE.Mesh(guardGeo, goldMaterial);
            guard.position.y = 0;
            swordGroup.add(guard);

            // 6.3 ด้ามจับ (Hilt/Handle)
            const handleGeo = new THREE.CylinderGeometry(0.06, 0.07, 1.0, 16);
            const handle = new THREE.Mesh(handleGeo, handleMaterial);
            handle.position.y = -0.55;
            swordGroup.add(handle);

            // 6.4 หัวด้ามจับ (Pommel)
            const pommelGeo = new THREE.SphereGeometry(0.12, 16, 16);
            const pommel = new THREE.Mesh(pommelGeo, goldMaterial);
            pommel.position.y = -1.1;
            swordGroup.add(pommel);

            // ปรับมุมตั้งต้นให้กระบี่เอียงสวยงาม
            swordGroup.rotation.z = -Math.PI / 4;
            return swordGroup;
        }

        const sword = createSword();
        scene.add(sword);

        // 7. สร้างอนุภาคแสงรอบๆ กระบี่ (Magic Particles)
        const particleCount = 150;
        const particleGeo = new THREE.BufferGeometry();
        const particlePos = new Float32Array(particleCount * 3);

        for (let i = 0; i < particleCount * 3; i += 3) {
            particlePos[i] = (Math.random() - 0.5) * 15;
            particlePos[i + 1] = (Math.random() - 0.5) * 15;
            particlePos[i + 2] = (Math.random() - 0.5) * 15;
        }

        particleGeo.setAttribute('position', new THREE.BufferAttribute(particlePos, 3));
        const particleMat = new THREE.PointsMaterial({
            color: 0x00ffff,
            size: 0.08,
            transparent: true,
            opacity: 0.7,
            blending: THREE.AdditiveBlending
        });
        const particles = new THREE.Points(particleGeo, particleMat);
        scene.add(particles);

        // 8. Grid Helper และ Ring อักขระเวทมนตร์ใต้พื้น
        const grid = new THREE.GridHelper(20, 20, 0x00f0ff, 0x112244);
        grid.position.y = -3;
        scene.add(grid);

        // 9. Interactive - ระบบคลิกกระบี่ (Raycasting)
        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();
        let isSpinningFast = false;

        window.addEventListener('click', (event) => {
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(sword.children);

            if (intersects.length > 0) {
                // เอฟเฟกต์เมื่อคลิกโดนกระบี่
                isSpinningFast = true;
                swordLight.intensity = 15;
                swordLight.color.setHex(0xff0055); // เปลี่ยนเป็นสีชมพูเรืองแสงชั่วคราว
                
                setTimeout(() => {
                    isSpinningFast = false;
                    swordLight.intensity = 3;
                    swordLight.color.setHex(0x00f0ff);
                }, 1200);
            }
        });

        // 10. Window Resize Handler
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        });

        // 11. Animation Loop
        const clock = new THREE.Clock();

        const animate = () => {
            requestAnimationFrame(animate);

            const elapsedTime = clock.getElapsedTime();

            // อัปเดตตำแหน่งกระบี่ให้บินหมุนวนไปมาเป็นคลื่น (Floating & Flying Path)
            const radius = 2.5;
            const speed = isSpinningFast ? 4 : 1;

            sword.position.x = Math.sin(elapsedTime * speed) * radius;
            sword.position.y = Math.sin(elapsedTime * 2 * speed) * 0.8 + 0.5;
            sword.position.z = Math.cos(elapsedTime * speed) * radius;

            // หมุนตัวกระบี่ตลอดเวลา
            sword.rotation.y += isSpinningFast ? 0.3 : 0.02;
            sword.rotation.x = Math.sin(elapsedTime * 2) * 0.2;

            // ให้แสงติดไปกับตัวกระบี่
            swordLight.position.copy(sword.position);

            // หมุนอนุภาคแสงรอบๆ
            particles.rotation.y = elapsedTime * 0.05;

            // อัปเดต Controls และ Render
            controls.update();
            renderer.render(scene, camera);
        };

        animate();
    </script>
</body>
</html>
