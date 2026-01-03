import * as THREE from 'three';
import { HandLandmarker, FilesetResolver } from '@mediapipe/tasks-vision';

// --- INITIALIZATION ---
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// --- PARTICLE SYSTEM ---
const particleCount = 5000;
const geometry = new THREE.BufferGeometry();
const positions = new Float32Array(particleCount * 3);
const colors = new Float32Array(particleCount * 3);

// Initial state: random sphere
for (let i = 0; i < particleCount * 3; i++) {
    positions[i] = (Math.random() - 0.5) * 10;
}

geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

const material = new THREE.PointsMaterial({
    size: 0.05,
    vertexColors: true,
    transparent: true,
    blending: THREE.AdditiveBlending
});

const particleSystem = new THREE.Points(geometry, material);
scene.add(particleSystem);

// --- SHAPE TEMPLATES ---
const shapes = {
    heart: (i) => {
        const t = Math.random() * Math.PI * 2;
        return {
            x: 16 * Math.sin(t) ** 3,
            y: 13 * Math.cos(t) - 5 * Math.cos(2*t) - 2 * Math.cos(3*t) - Math.cos(4*t),
            z: (Math.random() - 0.5) * 5
        };
    },
    saturn: (i) => {
        // Core sphere + flat ring
        if (i < particleCount * 0.5) {
            const phi = Math.random() * Math.PI * 2;
            const costheta = Math.random() * 2 - 1;
            const u = Math.random();
            const theta = Math.acos(costheta);
            const r = 3 * Math.cbrt(u);
            return { x: r * Math.sin(theta) * Math.cos(phi), y: r * Math.sin(theta) * Math.sin(phi), z: r * Math.cos(theta) };
        } else {
            const angle = Math.random() * Math.PI * 2;
            const r = 5 + Math.random() * 2;
            return { x: r * Math.cos(angle), y: (Math.random() - 0.5), z: r * Math.sin(angle) };
        }
    }
};

// --- HAND TRACKING LOGIC ---
async function setupHandTracking() {
    const vision = await FilesetResolver.forVisionTasks("https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest/wasm");
    const handLandmarker = await HandLandmarker.createFromOptions(vision, {
        baseOptions: { modelAssetPath: "path_to_hand_landmarker.task" },
        runningMode: "VIDEO"
    });

    // In the animation loop:
    // 1. Get landmarks from webcam video
    // 2. Map Landmark 8 (Index Tip) to Three.js coordinates
    // 3. If Distance(Landmark 4, Landmark 8) > threshold -> Trigger Expansion
}

function animate() {
    requestAnimationFrame(animate);
    // Add logic to lerp particle positions toward current template
    particleSystem.rotation.y += 0.005; 
    renderer.render(scene, camera);
}
animate();
