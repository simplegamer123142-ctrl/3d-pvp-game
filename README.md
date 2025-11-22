# 3d-pvp-game
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Arena Brawl</title>
    <!-- Tailwind CSS for utility classes -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Inter Font -->
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            margin: 0;
            overflow: hidden; /* Hide scrollbars */
            background-color: #1a202c; /* Dark background */
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
        }
        #game-container {
            position: relative;
            width: 90vw;
            height: 70vh;
            max-width: 1000px;
            max-height: 800px;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
            margin-bottom: 1rem;
            cursor: default; 
        }
        canvas {
            display: block;
            cursor: pointer; /* Indicate the canvas is clickable for FPP and actions */
        }
        .ui-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 1rem;
            display: flex;
            justify-content: space-between;
            pointer-events: none; /* Allows mouse events to pass through to the canvas */
            z-index: 10;
        }
        .bar-container {
            width: 120px; /* Slightly smaller for two bars */
            height: 20px;
            background-color: #333;
            border: 2px solid white;
            border-radius: 4px;
            overflow: hidden;
            box-shadow: 0 0 8px rgba(255, 255, 255, 0.3);
            margin-left: 0.5rem;
        }
        .health-bar, .energy-bar {
            height: 100%;
            transition: width 0.3s ease-out;
        }
        .fpp-status {
            position: absolute;
            bottom: 1rem;
            left: 50%;
            transform: translateX(-50%);
            padding: 0.5rem 1rem;
            background-color: rgba(255, 255, 0, 0.8);
            color: #1a202c;
            font-weight: bold;
            border-radius: 8px;
            transition: opacity 0.3s;
            z-index: 10;
            pointer-events: none;
        }
    </style>
    <!-- Three.js CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <h1 class="text-3xl font-bold text-white mb-4">3D Arena Brawl (vs AI)</h1>

    <div id="game-container">
        <!-- The UI overlay for health bars -->
        <div class="ui-overlay">
            <!-- Player 1 Health & Energy -->
            <div class="flex flex-col space-y-2 p-2 bg-gray-900/70 rounded-lg shadow-xl">
                <div class="flex items-center">
                    <span class="text-sm font-bold text-red-400 w-12">Health</span>
                    <div class="bar-container border-red-400">
                        <div id="p1-health" class="health-bar bg-red-600" style="width: 100%;"></div>
                    </div>
                </div>
                <div class="flex items-center">
                    <span class="text-sm font-bold text-yellow-400 w-12">Energy</span>
                    <div class="bar-container border-yellow-400">
                        <div id="p1-energy" class="energy-bar bg-yellow-500" style="width: 100%;"></div>
                    </div>
                </div>
                <span class="text-xl font-bold text-red-400 pt-1 text-center">YOU (P1)</span>
            </div>

            <!-- Player 2 Health -->
            <div class="flex flex-col items-end space-y-2 p-2 bg-gray-900/70 rounded-lg shadow-xl">
                <div class="flex items-center">
                    <div class="bar-container">
                        <div id="p2-health" class="health-bar bg-blue-600" style="width: 100%;"></div>
                    </div>
                    <span class="text-sm font-bold text-blue-400 w-12 text-right">Health</span>
                </div>
                <span class="text-xl font-bold text-blue-400 pt-1 text-center">AI (P2)</span>
            </div>
        </div>

        <div id="canvas-wrapper">
            <!-- Three.js renderer will append the canvas here -->
        </div>

        <!-- FPP Status Message -->
        <div id="fpp-status" class="fpp-status opacity-0">
            First-Person View Active (Mouse Look)
        </div>

        <!-- Message Box -->
        <div id="message-box" class="absolute inset-0 flex items-center justify-center backdrop-blur-sm hidden">
            <div class="bg-gray-800 text-white p-6 rounded-xl shadow-2xl text-center border-4 border-yellow-500 w-full max-w-sm">
                <h2 id="message-title" class="text-3xl font-extrabold mb-4 text-yellow-300">KO!</h2>
                <p id="message-text" class="text-lg mb-6"></p>
                <button id="restart-button" class="px-6 py-3 bg-green-600 hover:bg-green-700 text-white font-bold rounded-lg transition transform hover:scale-105">
                    Restart Game
                </button>
            </div>
        </div>

    </div>

    <!-- Controls & Instructions -->
    <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-8 text-white p-4 bg-gray-800/80 rounded-xl shadow-2xl">
        <div class="text-center">
            <h3 class="text-xl font-semibold text-red-400 mb-2">Player Controls</h3>
            <ul class="text-left space-y-1 text-sm">
                <li><span class="font-mono bg-gray-900 px-2 py-1 rounded">W/A/S/D</span>: Move</li>
                <li class="font-bold text-yellow-400"><span class="font-mono bg-gray-900 px-2 py-1 rounded">Left Click</span>: Punch (Costs 20 Energy)</li>
                <li class="font-bold text-green-400"><span class="font-mono bg-gray-900 px-2 py-1 rounded">Space</span>: Jump</li>
                <li class="font-bold text-yellow-400">
                    <span class="font-mono bg-gray-900 px-2 py-1 rounded">Ctrl</span>: Press to Toggle First-Person View (FPP)
                </li>
                <li class="font-bold text-yellow-400">
                    <span class="font-mono bg-gray-900 px-2 py-1 rounded">Mouse</span>: Controls Look Direction (<span class="text-green-300">Horizontal & Vertical</span>) in FPP
                </li>
            </ul>
        </div>
        <div class="text-center">
            <h3 class="text-xl font-semibold text-blue-400 mb-2">AI Opponent (Blue)</h3>
            <p class="text-sm text-yellow-500 font-bold">Arena is now 2x larger!</p>
            <p class="text-sm">Player stats have been boosted for a better experience!</p>
        </div>
    </div>


    <script type="module">
        // Three.js Variables
        let scene, camera, renderer, container;
        let p1, p2;
        let p1HealthElement, p2HealthElement, p1EnergyElement; 
        let messageBox, restartButton, messageTitle, messageText, fppStatus;

        // Game Variables (BOOSTED STATS FOR PLAYER)
        const MAX_HEALTH = 200; 
        const MAX_ENERGY = 100; 
        const ATTACK_COST = 20; 
        const ENERGY_REGEN_RATE = 1.0; 

        const PLAYER_SPEED = 0.8; 
        const ARENA_RADIUS = 40; 
        const ATTACK_RANGE = 4;
        const DAMAGE_AMOUNT = 30; 
        
        // JUMPING AND GRAVITY MECHANICS
        let isJumping = false;
        let jumpVelocity = 0;
        const JUMP_POWER = 1.0; 
        const GRAVITY = 0.05; 
        
        // AI Variables
        const AI_SPEED = 0.45; 
        let p2AttackCooldown = 0;
        const AI_COOLDOWN_TIME = 30;

        let p1Health = MAX_HEALTH; 
        let p1Energy = MAX_ENERGY; 
        let p2Health = 100;
        let isGameOver = false;

        // Camera Variables for Third-Person View
        const CAMERA_DISTANCE = 15; 
        const CAMERA_HEIGHT = 10;   
        const CAMERA_SMOOTHING = 0.2; 
        const FPP_SMOOTHING = 0.4;    
        const ROTATION_SMOOTHING = 0.1; 
        let cameraFollowAngle = 0; 
        let cameraPitch = 0; // NEW: Vertical rotation for FPP (look up/down)
        const FPP_HEIGHT = 6.0; // The height of the eyes above ground (Y=0)

        // Input Handling
        const keys = {};
        let isFirstPersonMode = false; 
        let isFppToggled = false; 
        const MOUSE_SENSITIVITY = 0.003; 
        let animationTime = 0; 
        const LOOK_AHEAD_DISTANCE = 50; // Distance used for calculating the FPP look vector

        // --- Utility Functions ---

        // Convert degrees to radians
        const toRadians = (degrees) => degrees * (Math.PI / 180);

        // Calculate distance between two objects on the XZ plane
        const getDistance = (obj1, obj2) => {
            const dx = obj1.position.x - obj2.position.x;
            const dz = obj1.position.z - obj2.position.z;
            return Math.sqrt(dx * dx + dz * dz);
        };

        // Handles the message box display (instead of alert/confirm)
        const showMessage = (title, text) => {
            isGameOver = true;
            messageTitle.textContent = title;
            messageText.textContent = text;
            messageBox.classList.remove('hidden');
        };

        // Updates the energy bar width
        function updateP1EnergyDisplay() {
            p1EnergyElement.style.width = `${(p1Energy / MAX_ENERGY) * 100}%`;
        }
        
        // Handles the procedural arm and leg swinging animation
        function handleLimbAnimation(fighter, isMoving) {
            if (!fighter.userData.leftArm) return; 

            const parts = fighter.userData;
            const speed = isMoving ? 1.0 : 0.2; 
            const amplitude = isMoving ? 0.6 : 0.05; 
            const bobAmplitude = isMoving ? 0.15 : 0.03;
            const bobFrequency = isMoving ? 0.15 : 0.05;
            
            // --- Limb Swing (Rotation around the X-axis) ---
            
            const armRotation = Math.sin(animationTime * 0.2 * speed) * amplitude;
            const legRotation = Math.cos(animationTime * 0.2 * speed) * amplitude;

            // Arms: Right Arm forward, Left Arm backward (and vice-versa)
            parts.rightArm.rotation.x = armRotation;
            parts.leftArm.rotation.x = -armRotation;
            
            // Legs: Right Leg backward, Left Leg forward
            parts.rightLeg.rotation.x = -legRotation;
            parts.leftLeg.rotation.x = legRotation;

            // --- Body Bob (Y position) ---
            fighter.position.y += Math.sin(animationTime * bobFrequency) * bobAmplitude;
        }


        // Creates a stylized fighter model (RIGGED for animation)
        function createBrawlFighter(color) {
            const material = new THREE.MeshStandardMaterial({ 
                color: color, 
                metalness: 0.2, 
                roughness: 0.6
            });

            const fighterGroup = new THREE.Group();
            
            // --- Dimensions ---
            const bodyHeight = 2.5;
            const bodyWidth = 2.0;
            const headRadius = 1.0;
            const hipHeight = 0.5;
            const legHeight = 2.0;
            const legRadius = 0.5;
            const armLength = 2.5;
            const armThickness = 0.4;
            
            // Calculate key Y positions (all relative to the ground Y=0)
            const legCenterY = legHeight / 2; 
            const hipCenterY = legHeight + hipHeight / 2; 
            const torsoCenterY = legHeight + hipHeight + bodyHeight / 2; 
            const headY = legHeight + hipHeight + bodyHeight + headRadius; 
            const armAttachmentY = legHeight + hipHeight + bodyHeight * 0.8; 

            // 1. Torso 
            const torsoGeometry = new THREE.CylinderGeometry(bodyWidth * 0.7, bodyWidth, bodyHeight, 8);
            const torso = new THREE.Mesh(torsoGeometry, material);
            torso.position.y = torsoCenterY; 
            torso.castShadow = true;
            fighterGroup.add(torso);
            
            // 2. Hips 
            const hipGeometry = new THREE.BoxGeometry(bodyWidth, hipHeight, 1.0);
            const hips = new THREE.Mesh(hipGeometry, material);
            hips.position.y = hipCenterY; 
            hips.castShadow = true;
            fighterGroup.add(hips);
            
            // 3. Legs (Create pivots for animation)
            const legGeometry = new THREE.CylinderGeometry(legRadius, legRadius, legHeight, 8);
            
            const leftLegMesh = new THREE.Mesh(legGeometry, material);
            const leftLeg = new THREE.Group();
            leftLeg.position.set(-bodyWidth / 2 + legRadius, legHeight, 0); // Pivot point at hip
            leftLegMesh.position.y = -legHeight / 2; // Mesh hangs down from pivot
            leftLeg.add(leftLegMesh);
            leftLegMesh.castShadow = true;
            fighterGroup.add(leftLeg);
            
            const rightLegMesh = new THREE.Mesh(legGeometry, material);
            const rightLeg = new THREE.Group();
            rightLeg.position.set(bodyWidth / 2 - legRadius, legHeight, 0); // Pivot point at hip
            rightLegMesh.position.y = -legHeight / 2; // Mesh hangs down from pivot
            rightLeg.add(rightLegMesh);
            rightLegMesh.castShadow = true;
            fighterGroup.add(rightLeg);

            // 4. Head
            const headGeometry = new THREE.SphereGeometry(headRadius, 16, 16);
            const head = new THREE.Mesh(headGeometry, material);
            head.position.y = headY; 
            head.castShadow = true;
            fighterGroup.add(head);

            // 5. Arms (Create pivots for animation)
            const armGeometry = new THREE.BoxGeometry(armThickness, armLength, armThickness);
            
            // Left Arm
            const leftArmMesh = new THREE.Mesh(armGeometry, material);
            const leftArm = new THREE.Group();
            leftArm.position.set(bodyWidth * 0.7 + armThickness * 0.5, armAttachmentY, 0); // Pivot point at shoulder
            leftArmMesh.position.y = -armLength / 2; // Mesh hangs down from pivot
            leftArmMesh.rotation.z = toRadians(10); // Default relaxed rotation
            leftArm.add(leftArmMesh);
            leftArmMesh.castShadow = true;
            fighterGroup.add(leftArm);
            
            // Right Arm
            const rightArmMesh = new THREE.Mesh(armGeometry, material);
            const rightArm = new THREE.Group();
            rightArm.position.set(-(bodyWidth * 0.7 + armThickness * 0.5), armAttachmentY, 0); // Pivot point at shoulder
            rightArmMesh.position.y = -armLength / 2; // Mesh hangs down from pivot
            rightArmMesh.rotation.z = toRadians(-10); // Default relaxed rotation
            rightArm.add(rightArmMesh);
            rightArmMesh.castShadow = true;
            fighterGroup.add(rightArm);
            
            // Store the rotatable groups for animation
            fighterGroup.userData.leftArm = leftArm;
            fighterGroup.userData.rightArm = rightArm;
            fighterGroup.userData.leftLeg = leftLeg;
            fighterGroup.userData.rightLeg = rightLeg;

            return fighterGroup;
        }


        // --- Core Three.js Setup ---

        function init() {
            // DOM element references
            container = document.getElementById('game-container');
            p1HealthElement = document.getElementById('p1-health');
            p2HealthElement = document.getElementById('p2-health');
            p1EnergyElement = document.getElementById('p1-energy'); 
            messageBox = document.getElementById('message-box');
            restartButton = document.getElementById('restart-button');
            messageTitle = document.getElementById('message-title');
            messageText = document.getElementById('message-text');
            fppStatus = document.getElementById('fpp-status');

            // 1. Scene Setup
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x10171d); 

            // 2. Camera Setup
            const aspectRatio = container.clientWidth / container.clientHeight;
            camera = new THREE.PerspectiveCamera(75, aspectRatio, 0.1, 1000);
            camera.position.set(0, 30, 30);

            // 3. Renderer Setup
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(container.clientWidth, container.clientHeight);
            renderer.shadowMap.enabled = true; 
            const canvasWrapper = document.getElementById('canvas-wrapper');
            canvasWrapper.appendChild(renderer.domElement);

            // 4. Create Game Objects
            createArena();
            createFighters();
            createLighting();

            // 5. Event Listeners
            window.addEventListener('resize', onWindowResize, false);
            document.addEventListener('keydown', onKeyDown, false);
            document.addEventListener('keyup', onKeyUp, false);
            renderer.domElement.addEventListener('mousedown', onMouseDown, false); // Left-Click for Punch
            restartButton.addEventListener('click', restartGame, false);
            
            // --- Pointer Lock Setup ---
            
            // 1. Add error handler for pointer lock issues
            renderer.domElement.addEventListener('pointerlockerror', (e) => {
                console.warn("Pointer Lock Error: Lock requested, but failed for document security reasons (expected in some iframes).", e);
                isFppToggled = false;
                updateFppState();
            }, false);
            
            // 2. Request lock on click, targeting the canvas element itself
            renderer.domElement.addEventListener('click', () => {
                if (!isGameOver && !document.pointerLockElement) {
                    renderer.domElement.requestPointerLock()
                        .catch(err => {
                            console.warn("Pointer Lock Request Failed (Handled):", err.name);
                            isFppToggled = false;
                            updateFppState();
                        });
                }
            }, false);
            
            // 3. Mouse move and state change handlers
            document.addEventListener('mousemove', onMouseMove, false);
            document.addEventListener('pointerlockchange', onPointerLockChange, false);
            
            // Initialize the health bars
            p1HealthElement.style.width = '100%';
            p2HealthElement.style.width = '100%';
        }

        // --- Object Creation Functions (Defined only once) ---

        function createArena() {
            // Ground Plane (Arena)
            const geometry = new THREE.CylinderGeometry(ARENA_RADIUS, ARENA_RADIUS, 1, 32);
            const material = new THREE.MeshPhongMaterial({ color: 0x4a5568, shininess: 50 });
            const arena = new THREE.Mesh(geometry, material);
            arena.receiveShadow = true;
            arena.position.y = -0.5; 
            scene.add(arena);

            // Boundary Visual (optional ring)
            const ringGeom = new THREE.RingGeometry(ARENA_RADIUS, ARENA_RADIUS + 0.5, 64);
            const ringMat = new THREE.MeshBasicMaterial({ color: 0xcccccc, side: THREE.DoubleSide });
            const ring = new THREE.Mesh(ringGeom, ringMat);
            ring.rotation.x = -Math.PI / 2;
            ring.position.y = 0.01;
            scene.add(ring);
        }

        function createFighters() {
            // Updated starting positions for larger arena
            p1 = createBrawlFighter(0xe53e3e);
            p1.position.set(-20, 0, 0); 
            p1.rotation.y = Math.PI / 2; 
            scene.add(p1);

            p2 = createBrawlFighter(0x3182ce);
            p2.position.set(20, 0, 0); 
            p2.rotation.y = -Math.PI / 2; 
            scene.add(p2);
        }

        function createLighting() {
            const ambientLight = new THREE.AmbientLight(0x404040, 2);
            scene.add(ambientLight);

            const directionalLight = new THREE.DirectionalLight(0xffffff, 1.5);
            directionalLight.position.set(15, 30, 15);
            directionalLight.castShadow = true;

            directionalLight.shadow.mapSize.width = 1024;
            directionalLight.shadow.mapSize.height = 1024;
            directionalLight.shadow.camera.near = 0.5;
            directionalLight.shadow.camera.far = 100;
            directionalLight.shadow.camera.left = -30;
            directionalLight.shadow.camera.right = 30;
            directionalLight.shadow.camera.top = 30;
            directionalLight.shadow.camera.bottom = -30;

            scene.add(directionalLight);
        }

        // --- Event Handlers & FPP State Management ---

        function onWindowResize() {
            const width = container.clientWidth;
            const height = container.clientHeight;

            camera.aspect = width / height;
            camera.updateProjectionMatrix();
            renderer.setSize(width, height);
        }
        
        function updateFppState() {
            const isLocked = !!document.pointerLockElement; 
            const shouldBeFPP = isFppToggled && isLocked;

            if (isFirstPersonMode !== shouldBeFPP) {
                isFirstPersonMode = shouldBeFPP;
                p1.visible = !isFirstPersonMode; 
                fppStatus.style.opacity = isFirstPersonMode ? 1 : 0;
                
                // Reset pitch when exiting FPP
                if (!isFirstPersonMode) {
                    cameraPitch = 0;
                }
            }
        }

        function onPointerLockChange() {
            updateFppState();
        }

        function onMouseMove(event) {
            if (isFirstPersonMode) {
                // Horizontal Look (Yaw)
                p1.rotation.y -= event.movementX * MOUSE_SENSITIVITY;
                
                // Vertical Look (Pitch) - Note: Positive movementY (down) should decrease cameraPitch (look down)
                cameraPitch -= event.movementY * MOUSE_SENSITIVITY;
                
                // Clamp pitch to prevent flipping (approx -89 to +89 degrees)
                const PI_HALF_CLAMPER = Math.PI / 2 - 0.01; 
                cameraPitch = Math.max(-PI_HALF_CLAMPER, Math.min(PI_HALF_CLAMPER, cameraPitch));
            }
        }
        
        // Handles Left-Click to punch
        function onMouseDown(event) {
            // event.button === 0 means Left Click
            if (!isGameOver && event.button === 0) {
                if (p1Energy >= ATTACK_COST) {
                    handleAttack(p1, p2, 1);
                } else {
                    console.log("Not enough energy to attack! (Requires 20 Energy)");
                }
            }
        }


        function onKeyDown(event) {
            keys[event.code] = true;
            
            if (!isGameOver) {
                // Jump Logic - only start jump if not already jumping
                if (event.code === 'Space' && !isJumping) {
                    isJumping = true;
                    jumpVelocity = JUMP_POWER; // Start the jump
                }
                
                if ((event.code === 'ControlLeft' || event.code === 'ControlRight')) {
                    const isLocked = !!document.pointerLockElement;
                    if (isLocked) {
                        isFppToggled = !isFppToggled; 
                        updateFppState(); 
                    }
                }
            }
        }

        function onKeyUp(event) {
            keys[event.code] = false;
        }


        // --- Game Logic Updates ---
        
        function handleJump() {
            if (isJumping) {
                // Apply velocity to Y position
                p1.position.y += jumpVelocity;
                
                // Apply gravity to velocity
                jumpVelocity -= GRAVITY;
                
                // Check if player has landed (Y position is at or below ground level)
                if (p1.position.y <= 0) {
                    p1.position.y = 0; // Snap to ground
                    isJumping = false;
                    jumpVelocity = 0;
                }
            }
        }

        function handleMovement() {
            const p1PrevPos = p1.position.clone();
            
            let isMoving = false; // Flag to check if any directional key is pressed

            // Controls: W (forward), S (backward), A (strafe left), D (strafe right)
            if (keys['KeyW']) { 
                p1.translateZ(PLAYER_SPEED);  // Move FORWARD 
                isMoving = true; 
            } 
            if (keys['KeyS']) { 
                p1.translateZ(-PLAYER_SPEED); // Move BACKWARD
                isMoving = true; 
            } 
            if (keys['KeyA']) { 
                p1.translateX(-PLAYER_SPEED); // Strafe Left
                isMoving = true; 
            } 
            if (keys['KeyD']) { 
                p1.translateX(PLAYER_SPEED);  // Strafe Right
                isMoving = true; 
            } 
            
            // Apply animation logic based on movement state
            handleLimbAnimation(p1, isMoving);

            // Keep fighter within the arena boundary
            checkBoundary(p1, p1PrevPos);
            
            // Handle vertical movement (Jumping/Gravity)
            handleJump();
        }
        
        function handleAI() {
            // P2 Animation: Make the AI shuffle and turn slightly
            handleLimbAnimation(p2, true); // Treat as 'always moving' for idle animation
            
            // To-Do: Implement better AI chase and attack logic
        }

        // Handles P1 energy regeneration
        function handleEnergyRegen() {
            if (p1Energy < MAX_ENERGY) {
                p1Energy = Math.min(MAX_ENERGY, p1Energy + ENERGY_REGEN_RATE);
                updateP1EnergyDisplay();
            }
        }

        // Handles the camera logic for TPP and FPP
        function updateCamera() {
            if (!p1) return;

            let targetCameraPosition = new THREE.Vector3();
            let targetLookAt = new THREE.Vector3();
            let smoothing = CAMERA_SMOOTHING;
            
            if (isFirstPersonMode) {
                // --- FIRST PERSON MODE (FPP) ---
                smoothing = FPP_SMOOTHING; 
                
                // Target position: On the player's head (eye level)
                targetCameraPosition.set(
                    p1.position.x, 
                    p1.position.y + FPP_HEIGHT, 
                    p1.position.z
                );
                
                // Target look-at calculation using spherical coordinates (Yaw and Pitch)
                const yaw = p1.rotation.y;
                const pitch = cameraPitch;
                const distance = LOOK_AHEAD_DISTANCE;
                
                // Calculate 3D direction vector components
                const dx = Math.sin(yaw) * Math.cos(pitch) * distance;
                const dy = Math.sin(pitch) * distance;
                const dz = Math.cos(yaw) * Math.cos(pitch) * distance;
                
                targetLookAt.set(
                    p1.position.x + dx,
                    p1.position.y + FPP_HEIGHT + dy,
                    p1.position.z + dz
                );
                
            } else {
                // --- THIRD PERSON MODE (TPP) ---
                
                // Smoothly update the follow angle to the player's last direction
                let targetAngle = p1.rotation.y;
                let diff = targetAngle - cameraFollowAngle;
                if (diff > Math.PI) diff -= (2 * Math.PI);
                if (diff < -Math.PI) diff += (2 * Math.PI);
                cameraFollowAngle += diff * ROTATION_SMOOTHING;
                cameraFollowAngle = cameraFollowAngle % (2 * Math.PI);

                // Calculate TPP position (behind and above the player)
                targetCameraPosition.set(
                    p1.position.x - Math.sin(cameraFollowAngle) * CAMERA_DISTANCE,
                    p1.position.y + CAMERA_HEIGHT,
                    p1.position.z - Math.cos(cameraFollowAngle) * CAMERA_DISTANCE
                );

                // Target look-at: Slightly above the player's center
                targetLookAt.set(p1.position.x, p1.position.y + 1, p1.position.z);
            }
            
            // Smoothly move the camera toward the target position
            camera.position.x += (targetCameraPosition.x - camera.position.x) * smoothing;
            camera.position.y += (targetCameraPosition.y - camera.position.y) * smoothing;
            camera.position.z += (targetCameraPosition.z - camera.position.z) * smoothing;

            // Update camera rotation to look at the target point
            camera.lookAt(targetLookAt);
        }


        function checkBoundary(fighter, prevPos) {
            const distanceToCenter = Math.sqrt(fighter.position.x * fighter.position.x + fighter.position.z * fighter.position.z);
            if (distanceToCenter > ARENA_RADIUS - 1) { 
                fighter.position.x = prevPos.x;
                fighter.position.z = prevPos.z;
            }
        }

        function handleAttack(attacker, target, playerNum) {
            const distance = getDistance(attacker, target);
            
            // Temporarily lift the arm for visual punch feedback
            const rightArm = attacker.userData.rightArm;
            if (rightArm) {
                rightArm.rotation.x = -Math.PI / 4; 
                setTimeout(() => {
                    rightArm.rotation.x = 0; // Reset after attack
                }, 100);
            }

            if (distance < ATTACK_RANGE) {
                
                if (playerNum === 1) {
                    p1Energy -= ATTACK_COST;
                    updateP1EnergyDisplay();
                }

                takeDamage(target, playerNum === 1 ? 2 : 1);
                
                // Visual feedback: brief color change on successful hit
                const torsoMesh = attacker.children.find(child => child.geometry.type === 'CylinderGeometry' && child.geometry.parameters.height === 2.5);
                
                if (torsoMesh) {
                    const originalColor = torsoMesh.material.color.getHex();
                    torsoMesh.material.color.set(0xffffff); 
                    setTimeout(() => {
                        torsoMesh.material.color.set(originalColor);
                    }, 100);
                }
            } else {
                if (playerNum === 1) {
                    p1Energy -= ATTACK_COST;
                    updateP1EnergyDisplay();
                }
                
                // Miss: brief visual feedback on attacker
                attacker.rotation.y += toRadians(20);
            }
        }

        function takeDamage(target, targetNum) {
            const isP1 = targetNum === 1;

            if (isP1) {
                p1Health -= DAMAGE_AMOUNT;
                if (p1Health < 0) p1Health = 0;
                p1HealthElement.style.width = `${(p1Health / MAX_HEALTH) * 100}%`;
                p1HealthElement.classList.add('transition-all');
                if (p1Health <= 0) {
                    showMessage("Defeat!", "Player 2 (AI) was victorious!");
                }
            } else {
                p2Health -= DAMAGE_AMOUNT;
                if (p2Health < 0) p2Health = 0;
                p2HealthElement.style.width = `${(p2Health / 100) * 100}%`;
                p2HealthElement.classList.add('transition-all');
                if (p2Health <= 0) {
                    showMessage("Victory!", "Player 1 (You) defeated the AI!");
                }
            }

            // Hit feedback (brief upward jump)
            // Note: If jumping, this overrides the jump position briefly, which is fine for hit stagger.
            target.position.y = Math.max(target.position.y, 1.0); 
            setTimeout(() => {
                // Only reset if not jumping, otherwise gravity takes over
                if (!isJumping && target.position.y > 0) {
                    target.position.y = 0; 
                }
            }, 50);
        }

        function restartGame() {
            p1Health = MAX_HEALTH; 
            p1Energy = MAX_ENERGY; 
            p2Health = 100;
            isGameOver = false;
            p2AttackCooldown = 0; 
            
            // Reset FPP state
            isFppToggled = false; 
            isFirstPersonMode = false;
            cameraPitch = 0; // Reset camera pitch
            if (document.pointerLockElement) {
                document.exitPointerLock();
            }

            p1HealthElement.style.width = '100%';
            p1EnergyElement.style.width = '100%';
            p2HealthElement.style.width = '100%';

            // Updated starting positions for larger arena
            p1.position.set(-20, 0, 0); 
            p2.position.set(20, 0, 0); 
            // Reset rotation to face each other
            p1.rotation.y = Math.PI / 2;
            p2.rotation.y = -Math.PI / 2;
            p1.visible = true; 
            fppStatus.style.opacity = 0;
            
            // Reset character bob/y position and animation time
            p1.position.y = 0;
            p2.position.y = 0;
            animationTime = 0; 
            
            // Reset jump state
            isJumping = false;
            jumpVelocity = 0;

            // Reset camera angle and position
            cameraFollowAngle = p1.rotation.y;
            updateCamera();

            messageBox.classList.add('hidden');
        }


        // --- Animation Loop ---

        function animate() {
            requestAnimationFrame(animate);

            if (!isGameOver) {
                animationTime++; 
                handleMovement(); 
                handleAI();     
                handleEnergyRegen(); 
                updateCamera(); 
            }

            renderer.render(scene, camera);
        }

        // Start the game when the window is fully loaded
        window.onload = function () {
            init();
            animate();
        };

    </script>

</body>
</html>
