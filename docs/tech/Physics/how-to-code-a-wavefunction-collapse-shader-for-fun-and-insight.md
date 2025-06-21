---
categories:
- Graphics
- Web Development
- Programming
comments: true
cover:
  image: https://images.pexels.com/photos/12356656/pexels-photo-12356656.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 14:26:03.015000
description: Dive into the fascinating world of Wavefunction Collapse (WFC) and learn
  how to implement its core principles in a GPU shader using WebGL. A practical, step-by-step
  guide for developers.
math: true
tags:
- WebGL
- GLSL
- Shaders
- Procedural Generation
- GPU
- Graphics
- JavaScript
- Front-end
title: How to Code a Wavefunction Collapse Shader for Fun and Insight
---

![Stunning blue and gold fractal art with intricate details in a vertical layout.](https://images.pexels.com/photos/12356656/pexels-photo-12356656.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Stunning blue and gold fractal art with intricate details in a vertical layout.")

## How to Code a Wavefunction Collapse Shader for Fun and Insight

The Wavefunction Collapse (WFC) algorithm is a fascinating concept in procedural generation, inspired by quantum mechanics. It's often used to generate levels, textures, or tilemaps by observing a small input sample and propagating constraints. While a full WFC *solver* is an iterative, stateful beast, we can embody its core principles – **entropy**, **observation**, and **propagation** – in a GPU shader for a visually compelling and insightful experience.

This post will guide you through building a simple WebGL shader that *simulates* WFC-like behavior. We won't be building a full-blown WFC solver on the GPU (that's a topic for an advanced compute shader series!), but rather a fragment shader that demonstrates the idea of "collapsing" a pixel's state from uncertainty to certainty, influenced by local conditions.

Let's dive in.

### Understanding Wavefunction Collapse (The Core Idea)

At its heart, WFC works like this:

1.  **Initial State (High Entropy):** Every "cell" (e.g., a tile in a grid) is initially in a superposition of all possible states. It "could be" any tile. This is its highest entropy state.
2.  **Observation (Collapse):** A cell is chosen (often the one with the lowest entropy – fewest remaining possibilities) and "observed" or "collapsed." This means picking one definitive state for it.
3.  **Propagation (Constraint Enforcement):** Once a cell's state is fixed, its neighbors' possible states are constrained. If Tile A can only be next to Tile B or C, then a neighbor of a collapsed Tile A can no longer be Tile D. This reduction of possibilities propagates through the system.
4.  **Iteration:** Repeat until all cells are collapsed, or a contradiction is reached (in which case, backtrack).

Translating this to a *shader* is tricky because shaders typically run in parallel, per-pixel, without direct knowledge of other pixels' *final* states in the same frame. Our approach will be to simulate the *visual effect* and *progression* of collapse over time, making pixels transition from an "uncertain" (noisy) state to a "certain" (fixed, patterned) state, with some influence from their spatial position or simple rules.

### Minimal Setup: HTML, JavaScript, and WebGL

To run our shader, we need a basic HTML page to host a `canvas` element and some JavaScript to set up the WebGL context, load and compile our shaders, and run a render loop.

We'll keep this as minimal as possible.

**1. `index.html`**

This file will contain our canvas and link to our JavaScript.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Wavefunction Collapse Shader</title>
    <style>
        body { margin: 0; overflow: hidden; background: #222; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
        canvas { display: block; border: 1px solid #444; }
    </style>
</head>
<body>
    <canvas id="glcanvas"></canvas>
    <script src="main.js"></script>
</body>
</html>
```

**2. `main.js`**

This JavaScript file will handle all the WebGL boilerplate.

```javascript
// main.js

let gl;
let program;
let startTime;

function main() {
    const canvas = document.getElementById('glcanvas');
    gl = canvas.getContext('webgl');

    if (!gl) {
        console.error('Unable to initialize WebGL. Your browser may not support it.');
        return;
    }

    // Set canvas dimensions to fill the window
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);

    const vertexShaderSource = `
        attribute vec4 a_position;
        void main() {
            gl_Position = a_position;
        }
    `;

    // We'll fill this with our WFC logic later
    const fragmentShaderSource = `
        precision highp float;
        uniform vec2 iResolution;
        uniform float iTime;

        void main() {
            vec2 uv = gl_FragCoord.xy / iResolution.xy;
            gl_FragColor = vec4(uv, 0.5 + 0.5 * sin(iTime), 1.0);
        }
    `;

    const vertexShader = compileShader(gl, vertexShaderSource, gl.VERTEX_SHADER);
    const fragmentShader = compileShader(gl, fragmentShaderSource, gl.FRAGMENT_SHADER);

    program = createProgram(gl, vertexShader, fragmentShader);
    gl.useProgram(program);

    // Create a buffer for a simple full-screen quad
    const positionBuffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
        -1, -1,
         1, -1,
        -1,  1,
        -1,  1,
         1, -1,
         1,  1,
    ]), gl.STATIC_DRAW);

    const positionAttributeLocation = gl.getAttribLocation(program, 'a_position');
    gl.enableVertexAttribArray(positionAttributeLocation);
    gl.vertexAttribPointer(positionAttributeLocation, 2, gl.FLOAT, false, 0, 0);

    startTime = Date.now();
    requestAnimationFrame(render);

    window.addEventListener('resize', () => {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);
        // Update iResolution uniform if needed, handled in render loop for simplicity
    });
}

function compileShader(gl, source, type) {
    const shader = gl.createShader(type);
    gl.shaderSource(shader, source);
    gl.compileShader(shader);

    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        console.error('Shader compilation error:', gl.getShaderInfoLog(shader));
        gl.deleteShader(shader);
        return null;
    }
    return shader;
}

function createProgram(gl, vertexShader, fragmentShader) {
    const program = gl.createProgram();
    gl.attachShader(program, vertexShader);
    gl.attachShader(program, fragmentShader);
    gl.linkProgram(program);

    if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
        console.error('Shader program linking error:', gl.getProgramInfoLog(program));
        gl.deleteProgram(program);
        return null;
    }
    return program;
}

function render() {
    const currentTime = (Date.now() - startTime) * 0.001; // Time in seconds

    const resolutionUniformLocation = gl.getUniformLocation(program, 'iResolution');
    gl.uniform2f(resolutionUniformLocation, gl.canvas.width, gl.canvas.height);

    const timeUniformLocation = gl.getUniformLocation(program, 'iTime');
    gl.uniform1f(timeUniformLocation, currentTime);

    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.TRIANGLES, 0, 6); // Draw the full-screen quad

    requestAnimationFrame(render);
}

window.onload = main;
```

**Output:**
Opening `index.html` in your browser will display a simple, animated gradient across the screen, confirming WebGL is working.

```output
(Browser window showing a full-screen canvas with a horizontally moving color gradient, confirming WebGL setup.)
```

### Step 1: Laying the Grid Foundation and Per-Cell Entropy

WFC operates on a grid of cells. Let's make our shader operate on a conceptual grid. Each cell will need its own unique "random" value – its initial entropy. We'll use a simple hashing function for this, derived from the cell's coordinates.

Update `main.js`'s `fragmentShaderSource` to the following:

```javascript
// Inside main.js, update fragmentShaderSource
const fragmentShaderSource = `
    precision highp float;
    uniform vec2 iResolution;
    uniform float iTime;

    // A simple 2D hash function for pseudo-random numbers
    // Based on The Art of Code's 'hash based on integer coords'
    float hash21(vec2 p) {
        p = fract(p * vec2(123.45, 678.90));
        p += dot(p, p + 45.67);
        return fract(p.x * p.y);
    }

    void main() {
        // Normalize screen coordinates to [0,1]
        vec2 uv = gl_FragCoord.xy / iResolution.xy;

        // Define our grid size (e.g., 20x20 cells)
        float gridSize = 20.0; // Number of cells across the screen

        // Calculate cell coordinates
        vec2 cellCoords = floor(uv * gridSize);

        // Get a unique pseudo-random value for this cell
        float cellEntropy = hash21(cellCoords);

        // Visualize entropy: lighter for higher entropy
        gl_FragColor = vec4(vec3(cellEntropy), 1.0);
    }
`;
```

**Output:**
You'll see a grid of randomly shaded gray squares. Each square has a unique, consistent shade of gray, representing its "entropy." This demonstrates that each cell has a unique identity, crucial for WFC.

```output
(Browser window showing a grid of uniformly sized squares, each with a distinct, static shade of gray. The shades appear random across the grid, but each square's shade remains constant.)
```

### Step 2: Simulating Collapse Over Time

Now, let's make these cells "collapse" from their entropic state to a fixed state. We'll use `iTime` to control this progression.

Each cell will transition from its initial noisy state to one of a few predefined "tiles" or "patterns." We'll simplify the "tiles" to just different colors for now.

Update `main.js`'s `fragmentShaderSource` again:

```javascript
// Inside main.js, update fragmentShaderSource
const fragmentShaderSource = `
    precision highp float;
    uniform vec2 iResolution;
    uniform float iTime;

    // A simple 2D hash function for pseudo-random numbers
    float hash21(vec2 p) {
        p = fract(p * vec2(123.45, 678.90));
        p += dot(p, p + 45.67);
        return fract(p.x * p.y);
    }

    // --- Tile definitions (simplified to colors) ---
    // These represent the possible "collapsed" states
    vec3 tileColorA = vec3(0.0, 0.5, 0.8); // Blue
    vec3 tileColorB = vec3(0.8, 0.2, 0.1); // Red
    vec3 tileColorC = vec3(0.1, 0.7, 0.3); // Green

    void main() {
        vec2 uv = gl_FragCoord.xy / iResolution.xy;
        float gridSize = 20.0;
        vec2 cellCoords = floor(uv * gridSize);
        vec2 cellCenter = (cellCoords + 0.5) / gridSize; // Center of the cell in UV space

        float cellEntropy = hash21(cellCoords);

        // Simulate collapse progress: ranges from 0 to 1
        // Faster near the center, slower at edges, and time-driven
        float collapseProgress = smoothstep(0.0, 1.0, (iTime * 0.1) + (1.0 - distance(uv, vec2(0.5))));

        // Determine the target state based on entropy
        vec3 targetColor;
        if (cellEntropy < 0.33) {
            targetColor = tileColorA;
        } else if (cellEntropy < 0.66) {
            targetColor = tileColorB;
        } else {
            targetColor = tileColorC;
        }

        // Interpolate between initial (noisy) state and collapsed (target) state
        vec3 initialColor = vec3(cellEntropy); // Represents the "uncertain" state
        vec3 finalColor = mix(initialColor, targetColor, collapseProgress);

        // Add a subtle grid line
        vec2 gridUV = fract(uv * gridSize);
        float border = smoothstep(0.01, 0.0, max(gridUV.x, gridUV.y) - 0.99); // Inner border
        border += smoothstep(0.01, 0.0, min(gridUV.x, gridUV.y));           // Outer border
        finalColor = mix(finalColor, vec3(0.05), border); // Darker border

        gl_FragColor = vec4(finalColor, 1.0);
    }
`;
```

**Explanation of changes:**
*   **Tile Colors:** We defined three `vec3` variables for distinct colors, representing our "collapsed" states.
*   **`collapseProgress`:** This uniform value controls how "collapsed" a cell is. We combine `iTime` with `distance(uv, vec2(0.5))` to make the collapse appear to originate from the center of the screen and spread outwards, creating a nice visual effect. `smoothstep` ensures a smooth transition.
*   **Target Color Selection:** Based on `cellEntropy`, each cell deterministically chooses its final color. This is its "observation."
*   **Interpolation:** `mix(initialColor, targetColor, collapseProgress)` smoothly blends between the initial noisy gray and the final chosen color as `collapseProgress` increases.
*   **Grid Lines:** Added a bit of code to draw subtle dark lines around cells for better visualization.

**Output:**
When you refresh, the grid will start as noisy gray squares. Over a few seconds, starting from the center and radiating outwards, each square will smoothly transition into one of the three defined colors (blue, red, or green), settling into its final, distinct state.

```output
(Browser window showing a grid of squares. Initially, all squares are shades of gray. As time progresses, squares in the center of the grid begin to transition into one of three distinct colors (blue, red, green). This transition spreads outwards, until the entire grid is composed of fixed colored squares.)
```

### Step 3: Introducing Simplified "Constraints" / Local Influence

The core of WFC is propagation: a collapsed cell influences its neighbors. In a single-pass fragment shader, we can't truly *propagate* states iteratively like the algorithm. However, we can *simulate* local influence by having a cell's "choice" be biased by its neighbors' *potential* states or by a central "seed" point.

Let's introduce a "seed" point (e.g., the exact center of the screen) that *always* collapses to a specific tile, and then make its neighbors more likely to pick certain states based on that. This isn't a true constraint propagation, but it visually demonstrates how a fixed point can influence its surroundings, mimicking WFC's spreading effect.

We'll add a function `getTileColor` that takes `cellCoords` and `cellEntropy` but also incorporates a simple rule: if a cell is near the center, it becomes `tileColorA`, otherwise it's chosen based on entropy.

```javascript
// Inside main.js, update fragmentShaderSource
const fragmentShaderSource = `
    precision highp float;
    uniform vec2 iResolution;
    uniform float iTime;

    // A simple 2D hash function for pseudo-random numbers
    float hash21(vec2 p) {
        p = fract(p * vec2(123.45, 678.90));
        p += dot(p, p + 45.67);
        return fract(p.x * p.y);
    }

    // --- Tile definitions (simplified to colors) ---
    vec3 tileColorA = vec3(0.0, 0.5, 0.8); // Blue (e.g., Water)
    vec3 tileColorB = vec3(0.8, 0.2, 0.1); // Red (e.g., Lava)
    vec3 tileColorC = vec3(0.1, 0.7, 0.3); // Green (e.g., Grass)
    vec3 tileColorD = vec3(0.7, 0.6, 0.2); // Brown (e.g., Dirt)

    // Function to determine the final tile color based on coordinates and entropy
    vec3 getTileColor(vec2 cellCoords, float cellEntropy, float gridSize) {
        // Define a "seed" point that forces a specific tile type
        vec2 centerCell = floor(gridSize / 2.0) - vec2(0.5, 0.5); // Center cell index
        
        // If within a small radius of the center, force Tile A (Water)
        if (distance(cellCoords, centerCell) < 1.5) { // Radius of 1.5 cells
            return tileColorA;
        }

        // If adjacent to a "water" tile, make it more likely to be grass/dirt
        // (This is a *very* simplified neighbor check, not true propagation)
        // Check 4 direct neighbors
        bool nearWater = false;
        if (distance(cellCoords + vec2(1,0), centerCell) < 1.5 ||
            distance(cellCoords + vec2(-1,0), centerCell) < 1.5 ||
            distance(cellCoords + vec2(0,1), centerCell) < 1.5 ||
            distance(cellCoords + vec2(0,-1), centerCell) < 1.5) {
            nearWater = true;
        }

        vec3 targetColor;
        if (nearWater) {
            // Near water, prefer grass or dirt
            if (cellEntropy < 0.5) {
                targetColor = tileColorC; // Grass
            } else {
                targetColor = tileColorD; // Dirt
            }
        } else {
            // Further away, use entropy for a mix of others
            if (cellEntropy < 0.3) {
                targetColor = tileColorB; // Lava
            } else if (cellEntropy < 0.6) {
                targetColor = tileColorC; // Grass
            } else {
                targetColor = tileColorD; // Dirt
            }
        }
        return targetColor;
    }

    void main() {
        vec2 uv = gl_FragCoord.xy / iResolution.xy;
        float gridSize = 30.0; // Increased grid density
        vec2 cellCoords = floor(uv * gridSize);
        vec2 cellCenterUV = (cellCoords + 0.5) / gridSize;

        float cellEntropy = hash21(cellCoords);

        // Determine the target state using our new function with simplified constraints
        vec3 targetColor = getTileColor(cellCoords, cellEntropy, gridSize);

        // Calculate collapse progress (still center-out, time-driven)
        // Adjust speed and range for a smoother or faster collapse
        float collapseFactor = 0.5; // Controls spread speed
        float timeOffset = iTime * collapseFactor;
        float distToCenter = distance(uv, vec2(0.5, 0.5));
        float collapseProgress = smoothstep(0.0, 1.0, timeOffset - (distToCenter * 2.0));

        // The initial "uncertain" state can be more dynamic (e.g., noisy static)
        vec3 initialColor = vec3(hash21(gl_FragCoord.xy * 0.1 + iTime * 5.0)); // Random noise changing over time

        vec3 finalColor = mix(initialColor, targetColor, clamp(collapseProgress, 0.0, 1.0));

        // Add a subtle grid line (remains the same)
        vec2 gridUV = fract(uv * gridSize);
        float border = smoothstep(0.01, 0.0, max(gridUV.x, gridUV.y) - 0.99);
        border += smoothstep(0.01, 0.0, min(gridUV.x, gridUV.y));
        finalColor = mix(finalColor, vec3(0.05), border);

        gl_FragColor = vec4(finalColor, 1.0);
    }
`;
```

**Key additions in this step:**
*   **More Tile Colors:** Added `tileColorD` (brown).
*   **`getTileColor` function:** This encapsulates our "decision-making" logic.
    *   **Forced Center:** If `cellCoords` are near the screen center, it *always* returns `tileColorA` (blue/water). This is our "observed" or "collapsed" seed.
    *   **Simplified "Neighbor" Check:** We check if the *current* cell is adjacent to where `tileColorA` *would be* placed. If so, it biases the current cell's choice towards `tileColorC` (green/grass) or `tileColorD` (brown/dirt). This is a very crude form of "propagation" by local proximity rules.
*   **Dynamic Initial State:** Changed `initialColor` to `vec3(hash21(gl_FragCoord.xy * 0.1 + iTime * 5.0))` for a more chaotic, "noisy static" appearance when uncollapsed.
*   **Adjusted `collapseProgress`:** Tweaked the formula slightly to control spread.

**Output:**
Now, when you refresh, you'll see a clear blue "water" patch form in the center of the grid, even as other cells are still collapsing. Around this blue patch, you'll predominantly see green ("grass") and brown ("dirt") tiles appearing as they collapse, with red ("lava") tiles appearing further out. This demonstrates how a fixed "observation" can influence the "collapse" of surrounding cells based on simple rules.

```output
(Browser window showing a grid of squares. Initially, all squares are dynamic noisy static. As time progresses, a distinct blue patch forms in the exact center. Around this blue patch, green and brown squares predominantly appear, while red squares are more common further away. The collapse animation still radiates outwards from the center.)
```

### Further Ideas and Improvements

This shader provides a basic visual representation. Here are some ways to expand upon it:

*   **More Complex Tiles:** Instead of just colors, use `mod` operations and `step` functions within `getTileColor` to draw simple patterns (e.g., roads, walls, grass textures) based on the chosen tile state. You could also sample from a texture atlas containing pre-drawn tiles.
*   **Input-Driven Collapse:** Instead of `iTime`, use `iMouse` to trigger collapses. When the mouse is clicked, set the closest cell to a specific state, and let the "propagation" (our simplified neighbor influence) spread from there.
*   **Multiple Seed Points:** Allow multiple fixed points to be "collapsed," and observe how their influences interact.
*   **Better Noise/Hashing:** For more organic results, explore Perlin noise or Simplex noise functions instead of simple hash functions.
*   **True Constraint Representation (Advanced):** For a more accurate WFC simulation, you'd need:
    *   **Multi-pass Rendering:** Render to textures (Frame Buffer Objects - FBOs). One pass to calculate entropy, another to observe a point, subsequent passes to propagate constraints by reading neighbor states from the previous pass's FBO.
    *   **State Encoding:** Each pixel's color (or separate textures) could encode bitmasks representing possible tile states. Propagation would involve ANDing these bitmasks with allowed neighbor patterns.
    *   **Compute Shaders:** For even more complex, iterative algorithms like WFC, WebGPU with its compute shaders would be a more suitable environment, allowing true parallel iteration and global memory access.

### Conclusion

While implementing a full-fledged Wavefunction Collapse *algorithm* directly in a single-pass fragment shader is impractical, we've successfully built a shader that visually demonstrates its core ideas:
*   **Entropy:** Represented by the initial noisy or random state of each cell.
*   **Observation/Collapse:** The transition of a cell from its uncertain state to a fixed, chosen state.
*   **Simplified Propagation/Constraints:** The influence of a "seed" point or simple proximity rules on neighbor cell choices, creating a structured, non-random output.

This project is a fantastic way to grasp the concepts of procedural generation, parallel processing on the GPU, and how complex ideas can be broken down and visualized. It's a testament to the power and flexibility of shaders, even for tasks they weren't originally designed for. Happy coding!