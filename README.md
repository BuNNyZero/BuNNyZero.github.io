# BuNNyZero.github.io

<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Flying Cat - Three.js</title>
    <style>
        html, body {
            width: 100%;
            height: 100%;
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #0b091a;
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

        .instructions {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            color: rgba(255, 255, 255, 0.85);
            font-size: 14px;
            pointer-events: none;
            user-select: none;
            text-align: center;
            background: rgba(0, 0, 0, 0.6);
            padding: 8px 18px;
            border-radius: 20px;
            backdrop-filter: blur(5px);
            border: 1px solid rgba(255, 255, 255, 0.15);
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
        🐾 คลิกซ้ายลากเพื่อหมุน | สโครลเพื่อซูม | คลิกที่ตัวน้องแมวเพื่อเพิ่มความเร็ว!
    </div>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        // 1. Scene Setup
        const canvas = document.querySelector('#webgl-canvas');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x0b091a, 0.025);

        // 2. Camera Setup
        const camera = new THREE.PerspectiveCamera(
            60, 
            window.innerWidth / window.innerHeight, 
            0.1, 
            1000
        );
        camera.position.set(0, 3, 9);

        // 3. Renderer Setup
        const renderer = new THREE.WebGLRenderer({
            canvas: canvas,
            antialias: true
        });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

        // 4. OrbitControls
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;

        // 5. Lighting Setup
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.9);
        scene.add(ambientLight);

        const mainLight = new THREE.DirectionalLight(0xffb703, 2);
        mainLight.position.set(5, 8, 5);
        scene.add(mainLight);

        const catLight = new THREE.PointLight(0xffaa00, 3, 7);
        scene.add(catLight);

        // 6. ฟังก์ชันสร้างโมเดลน้องแมว 3D (Procedural 3D Flying Cat)
        let tailMesh; // สำหรับสบัดหาง
        function createCat() {
            const catGroup = new THREE.Group();

            // Materials
            const furMaterial = new THREE.MeshStandardMaterial({
                color: 0x3a3a42, // แมวสีเทาเข้ม
                roughness: 0.8,
            });
            const innerEarMaterial = new THREE.MeshStandardMaterial({
                color: 0xffb3c1,
                roughness: 0.5
            });
            const eyeMaterial = new THREE.MeshBasicMaterial({
                color: 0x00ffcc // ตาเรืองแสงสีเขียวฟ้า
            });
            const noseMaterial = new THREE.MeshStandardMaterial({
                color: 0xff8fa3,
                roughness: 0.3
            });

            // 6.1 ลำตัว (Body)
            const bodyGeo = new THREE.SphereGeometry(0.8, 32, 16);
            bodyGeo.scale(1, 0.8, 1.4); // ยืดให้กลายเป็นทรงแคปซูล
            const body = new THREE.Mesh(bodyGeo, furMaterial);
            catGroup.add(body);

            // 6.2 หัว (Head)
            const headGeo = new THREE.SphereGeometry(0.65, 32, 16);
            headGeo.scale(1.1, 0.95, 1);
            const head = new THREE.Mesh(headGeo, furMaterial);
            head.position.set(0, 0.4, 0.9);
            catGroup.add(head);

            // 6.3 หูแมว (Ears)
            const earGeo = new THREE.ConeGeometry(0.22, 0.4, 16);
            
            const leftEar = new THREE.Mesh(earGeo, furMaterial);
            leftEar.position.set(0.32, 0.95, 0.85);
            leftEar.rotation.set(-0.2, 0, -0.2);
            catGroup.add(leftEar);

            const rightEar = leftEar.clone();
            rightEar.position.x = -0.32;
            rightEar.rotation.z = 0.2;
            catGroup.add(rightEar);

            // ด้านในหู
            const innerEarGeo = new THREE.ConeGeometry(0.14, 0.3, 16);
            const leftInnerEar = new THREE.Mesh(innerEarGeo, innerEarMaterial);
            leftInnerEar.position.set(0.32, 0.94, 0.88);
            leftInnerEar.rotation.set(-0.2, 0, -0.2);
            catGroup.add(leftInnerEar);

            const rightInnerEar = leftInnerEar.clone();
            rightInnerEar.position.x = -0.32;
            rightInnerEar.rotation.z = 0.2;
            catGroup.add(rightInnerEar);

            // 6.4 ตาแมว (Eyes)
            const eyeGeo = new THREE.SphereGeometry(0.09, 16, 16);
            const leftEye = new THREE.Mesh(eyeGeo, eyeMaterial);
            leftEye.position.set(0.22, 0.48, 1.48);
            catGroup.add(leftEye);

            const rightEye = leftEye.clone();
            rightEye.position.x = -0.22;
            catGroup.add(rightEye);

            // จมูก (Nose)
            const noseGeo = new THREE.SphereGeometry(0.05, 16, 16);
            const nose = new THREE.Mesh(noseGeo, noseMaterial);
            nose.position.set(0, 0.38, 1.53);
            catGroup.add(nose);

            // 6.5 ขาหน้า-ขาหลัง (Paws)
            const pawGeo = new THREE.SphereGeometry(0.2, 16, 16);
            pawGeo.scale(0.8, 1.2, 0.8);

            const FLPaw = new THREE.Mesh(pawGeo, furMaterial);
            FLPaw.position.set(0.4, -0.5, 0.6);
            catGroup.add(FLPaw);

            const FRPaw = FLPaw.clone();
            FRPaw.position.x = -0.4;
            catGroup.add(FRPaw);

            const BLPaw = FLPaw.clone();
            BLPaw.position.set(0.45, -0.4, -0.6);
            catGroup.add(BLPaw);

            const BRPaw = BLPaw.clone();
            BRPaw.position.x = -0.45;
            catGroup.add(BRPaw);

            // 6.6 หางแมว (Tail)
            const tailGeo = new THREE.CylinderGeometry(0.06, 0.04, 1.2, 16);
            tailGeo.translate(0, 0.6, 0); // ตั้งจุดหมุนที่โคนหาง
            tailMesh = new THREE.Mesh(tailGeo, furMaterial);
            tailMesh.position.set(0, 0, -1.1);
            tailMesh.rotation.x = -Math.PI / 3;
            catGroup.add(tailMesh);

            return catGroup;
        }

        const cat = createCat();
        scene.add(cat);

        // 7. อนุภาคแสงดาวรอบตัวแมว (Magic Particles)
        const particleCount = 200;
        const particleGeo = new THREE.BufferGeometry();
        const particlePos = new Float32Array(particleCount * 3);

        for (let i = 0; i < particleCount * 3; i += 3) {
            particlePos[i] = (Math.random() - 0.5) * 16;
            particlePos[i + 1] = (Math.random() - 0.5) * 16;
            particlePos[i + 2] = (Math.random() - 0.5) * 16;
        }

        particleGeo.setAttribute('position', new THREE.BufferAttribute(particlePos, 3));
        const particleMat = new THREE.PointsMaterial({
            color: 0xffbe0b,
            size: 0.09,
            transparent: true,
            opacity: 0.8,
            blending: THREE.AdditiveBlending
        });
        const particles = new THREE.Points(particleGeo, particleMat);
        scene.add(particles);

        // 8. Grid Helper พื้นหลัง
        const grid = new THREE.GridHelper(20, 20, 0xffa500, 0x331144);
        grid.position.y = -3;
        scene.add(grid);

        // 9. Interactive - ระบบคลิกตัวแมว (Raycasting)
        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();
        let isSuperSpeed = false;

        window.addEventListener('click', (event) => {
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(cat.children);

            if (intersects.length > 0) {
                // คลิกโดนแมว -> แมวเร่งสปีดควงสว่าง + เปลี่ยนสีแสงเรือง
                isSuperSpeed = true;
                catLight.intensity = 12;
                catLight.color.setHex(0xff0077);
                
                setTimeout(() => {
                    isSuperSpeed = false;
                    catLight.intensity = 3;
                    catLight.color.setHex(0xffaa00);
                }, 1500);
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
            const speed = isSuperSpeed ? 4 : 1.2;

            // 11.1 เส้นทางบินของน้องแมว (Trajectory)
            const radius = 2.8;
            cat.position.x = Math.sin(elapsedTime * speed) * radius;
            cat.position.y = Math.sin(elapsedTime * 2.5 * speed) * 0.6 + 0.5;
            cat.position.z = Math.cos(elapsedTime * speed) * radius;

            // 11.2 หมุนลำตัวตามทิศทางการบิน
            cat.rotation.y = elapsedTime * speed + Math.PI / 2;
            cat.rotation.z = Math.sin(elapsedTime * 3) * 0.15; // เอียงตัวตามจังหวะบิน
            
            if (isSuperSpeed) {
                cat.rotation.x += 0.3; // หมุนม้วนหน้าควงสว่าน
            } else {
                cat.rotation.x = 0;
            }

            // 11.3 สบัดหางแมวตามเวลา
            if (tailMesh) {
                tailMesh.rotation.z = Math.sin(elapsedTime * 6) * 0.3;
            }

            // แสงติดไปกับตัวแมว
            catLight.position.copy(cat.position);

            // หมุนอนุภาคแสงรอบๆ
            particles.rotation.y = elapsedTime * 0.08;

            controls.update();
            renderer.render(scene, camera);
        };

        animate();
    </script>
</body>
</html>
