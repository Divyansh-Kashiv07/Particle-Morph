# ✋ Interactive Hand-Controlled Particle System ✨

> An immersive web experiment that uses a combination of MediaPipe's Hand Tracking and Three.js's WebGL rendering to create a dynamic, user-controlled 3D particle system. Users can manipulate the shape, rotation, and color of thousands of glowing particles with real-time hand gestures.

## 🌟 Demo & Concept


This project merges computer vision (MediaPipe) with real-time 3D graphics (Three.js) to offer a novel, controller-free interactive experience. It showcases **morphing** between different geometric shapes defined by mathematical functions.

## 🚀 Technologies Used
* **Three.js (r128):** Main 3D graphics library for rendering the particle system.
* **WebGL:** The underlying API used by Three.js for hardware-accelerated rendering.
* **Custom GLSL Shaders:** Used to define the behavior and appearance of each particle, providing effects like glow and size scaling.
* **MediaPipe Hands:** A Google solution for real-time, high-fidelity hand and finger tracking using a webcam.
* **JavaScript (ES6):** Core logic for shape generation, gesture detection, and the animation loop.

## ✨ Features

### ✋ Real-Time Hand Control
The particle system responds directly to a single hand detected by the webcam:

| Gesture | Interaction | Technical Implementation |
| :--- | :--- | :--- |
| **Move Hand** (Open Palm) | **Rotation & Color Shift** | Hand's 2D screen position (X/Y) is mapped to the 3D particle mass rotation (`particles.rotation.x`, `particles.rotation.y`) and a dynamic HSL color shift (`uColor2` uniform). |
| **Pinch** (Thumb to Index Finger) | **Expand/Contract** | The distance between the thumb tip (landmark 4) and index finger tip (landmark 8) controls a custom shader uniform (`uExpansion`) to push the particles radially outwards. |
| **Fist** (Closing Hand) | **Switch Shape** | Detected by the average distance of finger tips (8, 12, 16, 20) to the wrist (landmark 0). Triggers a seamless morph to the next shape in the sequence. |

### 🖼️ Particle & Shape Generation
* **Morphing:** Particles smoothly interpolate (`THREE.MathUtils.lerp`) from their current position to the `targetPositions` of the next shape over time (`TRANSITION_SPEED`).
* **Custom Shapes:** Includes mathematically defined shapes:
    * `Heart` (Cardioid Formula)
    * `Saturn` (Sphere + Planar Rings)
    * `Flower` (Rose Curve)
    * `Fireworks` (Randomized 3D Sphere)
* **Glow Effect:** Achieved using a custom fragment shader, employing `THREE.AdditiveBlending` and depth-testing disabled (`depthWrite: false`) for a bright, energetic visual style.

## 🛠️ Setup

This project runs entirely in a modern web browser and only requires a webcam.

1.  **Save the Code:** Save the entire code block above as an HTML file (e.g., `index.html`).
2.  **Web Server Required:** Due to browser security restrictions on accessing the webcam and loading large models, you **must** run this file through a local web server (running it directly via `file:///...` path will likely fail).

### Recommended Setup (Using VS Code Live Server)

1.  Install **Visual Studio Code**.
2.  Install the **Live Server** extension (Ritwick Dey).
3.  Open the folder containing your `index.html` file in VS Code.
4.  Right-click on `index.html` and select **"Open with Live Server"**.

A browser window will open, and it will prompt you for webcam access.

## ⚙️ Key Code Snippets

### 1. The Morphing Mechanism (Animation Loop)

This loop provides the smooth transition between shapes:

```javascript
// Function: animate()
// ...
const positionsArr = particles.geometry.attributes.position.array;
// Morphing Logic (Vertex Interpolation)
for(let i = 0; i < PARTICLE_COUNT * 3; i++) {
    // Ease current position towards target position
    positionsArr[i] += (targetPositions[i] - positionsArr[i]) * TRANSITION_SPEED;
}
particles.geometry.attributes.position.needsUpdate = true;
// ...

The shader handles dynamic movement, pinch-based expansion, and particle sizing:

OpenGL Shading Language

// Vertex Shader snippet for expansion and noise
void main() {
    // ...
    vec3 pos = position;
    
    // Add some noise movement for dynamism
    pos.x += sin(uTime * 2.0 + position.y) * 0.1;
    pos.y += cos(uTime * 1.5 + position.x) * 0.1;

    // Expansion logic (based on pinch from MediaPipe)
    pos += normalize(pos) * uExpansion * 15.0;

    vec4 mvPosition = modelViewMatrix * vec4(pos, 1.0);
    gl_PointSize = size * (200.0 / -mvPosition.z) * uPixelRatio; // Perspective scaling
    gl_Position = projectionMatrix * mvPosition;
}
📜 License
This project is shared under the MIT License. Feel free to explore, modify, and reuse!