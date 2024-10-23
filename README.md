# Dynamic PBR Texture Resolution Switching for GLTF Models

This guide explains how to dynamically switch between different resolutions of PBR textures (Physically Based Rendering) on a GLTF model in a renderer like Three.js. This approach is ideal when you have multiple texture resolutions and want to optimize performance by using lower resolution textures when the model is far from the camera, or progressively load higher resolution textures when needed.

## Folder Structure

Organize your textures by material and resolution to ensure efficient management. Here's a suggested folder structure:

```
/textures
  ├── pbrmat1
  │   ├── 8x8
  │   │   ├── pbrmat1_albedo.png
  │   │   ├── pbrmat1_ao.png
  │   │   ├── pbrmat1_height.png
  │   │   ├── pbrmat1_normal.png
  │   │   └── pbrmat1_roughness.png
  │   ├── 16x16
  │   │   └── ...
  │   ├── 64x64
  │   │   └── ...
  │   ├── 1024x1024
  │   │   └── ...
```

## Steps

### 1. Load the GLTF Model

Use the `GLTFLoader` to load the GLTF model.

```js
const loader = new THREE.GLTFLoader();
loader.load('path/to/model.gltf', function(gltf) {
  const model = gltf.scene;
  scene.add(model);
});
```

### 2. Define Available Resolutions

Create a data structure that maps the different resolutions to their respective texture URLs.

```js
const textureResolutions = {
  '8x8': {
    albedo: 'textures/pbrmat1/8x8/pbrmat1_albedo.png',
    ao: 'textures/pbrmat1/8x8/pbrmat1_ao.png',
    normal: 'textures/pbrmat1/8x8/pbrmat1_normal.png',
    roughness: 'textures/pbrmat1/8x8/pbrmat1_roughness.png'
  },
  '1024x1024': {
    albedo: 'textures/pbrmat1/1024x1024/pbrmat1_albedo.png',
    ao: 'textures/pbrmat1/1024x1024/pbrmat1_ao.png',
    normal: 'textures/pbrmat1/1024x1024/pbrmat1_normal.png',
    roughness: 'textures/pbrmat1/1024x1024/pbrmat1_roughness.png'
  }
};
```

### 3. Create a Function to Switch Textures

Write a function that switches the model's textures based on the selected resolution.

```js
function switchTextures(model, resolution) {
  const textures = textureResolutions[resolution];
  const loader = new THREE.TextureLoader();

  model.traverse((child) => {
    if (child.isMesh && child.material) {
      child.material.map = loader.load(textures.albedo);
      child.material.aoMap = loader.load(textures.ao);
      child.material.normalMap = loader.load(textures.normal);
      child.material.roughnessMap = loader.load(textures.roughness);
      child.material.needsUpdate = true;  // Mark material for update
    }
  });
}
```

### 4. Trigger Texture Switching

Switch between different texture resolutions based on the camera distance or user settings.

```js
function updateTextureBasedOnCamera(model, camera) {
  const distance = camera.position.distanceTo(model.position);

  if (distance > 100) {
    switchTextures(model, '8x8');  // Use lower resolution for far distance
  } else {
    switchTextures(model, '1024x1024');  // Use high resolution for close distance
  }
}

function animate() {
  requestAnimationFrame(animate);
  updateTextureBasedOnCamera(model, camera);
  renderer.render(scene, camera);
}

animate();
```

### 5. Progressive Texture Loading

For better performance, you can progressively load higher resolution textures after the lower resolution ones are applied.

```js
// Load low resolution first
switchTextures(model, '8x8');

// Then load high resolution after some time
setTimeout(() => {
  switchTextures(model, '1024x1024');
}, 2000);  // Simulate delay for high-res texture loading
```

### 6. Memory Management

Dispose of textures that are no longer in use to free up memory.

```js
function disposeTexture(texture) {
  texture.dispose();
}
```

## Summary of Steps

1. **Organize Textures by Resolution**: Use a structured folder system to manage different texture resolutions.
2. **Load GLTF Model**: Load the model using Three.js and ensure the material supports PBR maps.
3. **Dynamic Texture Switching**: Implement logic to switch textures based on resolution using camera distance or user input.
4. **Progressive Loading**: Load lower resolution textures first, then progressively load higher ones for better performance.
5. **Memory Management**: Properly dispose of textures when they are no longer needed.

By following these steps, you can efficiently manage multiple resolutions of PBR textures and dynamically switch between them in a renderer for optimized performance.
