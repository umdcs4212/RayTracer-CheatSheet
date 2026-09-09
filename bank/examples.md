# Example Scenes - Parameters Reference

This page contains parameter configurations for example scenes you can use to test your ray tracer implementation at each stage of development.

---

## How to Create a Scene

To create a scene in your `fbMain.cpp`:

1. Set up a **Camera** with position, view direction, focal length, and image plane dimensions
2. Create **Lights** with positions and colors
3. Add **Shapes** (spheres, triangles) with positions, sizes, colors, and shaders
4. Compile and run: `cmake --build .` then `./src/fbMain`

---

## Camera Parameters

All cameras are `PerspectiveCamera` with the following parameters:

```
PerspectiveCamera(
  origin,              // vec3: camera position
  viewDir,             // vec3: direction camera is looking
  focal_length,        // float: distance to image plane
  image_plane_width,   // float: width of virtual image plane
  image_plane_height,  // float: height of virtual image plane
  pixel_nx,            // int: image width in pixels
  pixel_ny             // int: image height in pixels
)
```

### Example Camera Configurations

**Default Camera (Week 3):**
```cpp
PerspectiveCamera cam(
  vec3(0, 0, 0),       // origin at world center
  vec3(0, 0, -1),      // looking down -Z axis
  1.0,                 // focal length
  0.5,                 // image plane width
  0.5,                 // image plane height
  800,                 // 800×800 pixels
  800
);
```

**Wide View (Week 4-5):**
```cpp
PerspectiveCamera cam(
  vec3(0, 0, 0),
  vec3(0, 0, -1),
  1.0,
  2.0,                 // wider image plane
  2.0,
  400,
  400
);
```

**Custom Position (Week 5-6):**
```cpp
PerspectiveCamera cam(
  vec3(0, 3.0, 2.0),   // elevated position
  vec3(0, -1.5, -3.0), // looking down and forward
  0.4,
  0.6,
  0.6,
  800,
  800
);
```

---

## Light Parameters

All lights are `PointLight` with the following parameters:

```
PointLight(
  position,      // vec3: light position in world space
  color,         // vec3: light color (RGB, typically 0.0-1.0)
  intensity      // float: brightness multiplier (default 1.0)
)
```

### Example Light Configurations

**Single Light (Week 5):**
```cpp
lights.push_back(std::make_shared<PointLight>(
  vec3(2, 3, 1),        // position
  vec3(1.0, 1.0, 1.0)   // white color
));
```

**Two Lights (Week 6):**
```cpp
lights.push_back(std::make_shared<PointLight>(
  vec3(3, 5, 2),        // right light
  vec3(1.0, 1.0, 1.0)
));

lights.push_back(std::make_shared<PointLight>(
  vec3(-3, 5, 2),       // left light
  vec3(1.0, 1.0, 1.0)
));
```

---

## Shape Parameters

### Sphere

```
Sphere(
  center,        // vec3: sphere center position
  radius,        // float: sphere radius
  color,         // vec3: RGB color (0.0-1.0)
  shader         // std::shared_ptr<Shader>: material
)
```

**Example Spheres:**
```cpp
// Simple sphere (Week 4)
shapes.push_back(std::make_shared<Sphere>(
  vec3(0, 0, -10),    // center
  3.0f,               // radius
  vec3(0.149, 0.451, 0.698)  // blue
));

// With Lambertian shader (Week 5)
shapes.push_back(std::make_shared<Sphere>(
  vec3(-2.5, 1.0, -4.0),
  1.0f,
  vec3(0.0, 0.0, 1.0),        // blue
  lambertianShader
));

// With Mirror shader (Week 6)
shapes.push_back(std::make_shared<Sphere>(
  vec3(1.5, 1.10, -2.5),
  1.10f,
  vec3(0.8, 0.8, 0.8),        // light gray
  mirrorShader
));
```

### Triangle

```
Triangle(
  vertex_a,      // vec3: first vertex position
  vertex_b,      // vec3: second vertex position
  vertex_c,      // vec3: third vertex position
  color,         // vec3: RGB color (0.0-1.0)
  shader         // std::shared_ptr<Shader>: material (optional)
)
```

**Example Triangles:**
```cpp
// Simple triangle (Week 4)
shapes.push_back(std::make_shared<Triangle>(
  vec3(-1.2, -0.2, -7),
  vec3(0.8, -0.5, -5),
  vec3(0.9, 0, -5),
  vec3(1.0, 0.0, 0.0)    // red
));

// Ground plane (Week 6)
shapes.push_back(std::make_shared<Triangle>(
  vec3(0, 0, 5),
  vec3(200, 0, -200),
  vec3(-200, 0, -200),
  vec3(0.8, 0.8, 0.8),          // light gray
  diffuseGroundShader
));
```

---

## Shader Parameters

### LambertianShader
```cpp
auto shader = std::make_shared<LambertianShader>();
```

### BlinnPhongShader
```cpp
auto shader = std::make_shared<BlinnPhongShader>();
shader->setEyePosition(cam.getPosition());  // set eye position
```

### MirrorShader (Week 6)
```cpp
auto shader = std::make_shared<MirrorShader>();
```

### DiffuseShader (Week 6)
```cpp
auto shader = std::make_shared<DiffuseShader>(
  vec3(0.8, 0.8, 0.8)    // reflectance color
);
```

### NormalShader
```cpp
// Uses default constructor, no parameters
NormalShader normalShader;
```

---

## Quick Reference Table

| Component | Parameter | Type | Example |
|-----------|-----------|------|---------|
| **Camera** | origin | vec3 | `vec3(0, 3, 2)` |
| | viewDir | vec3 | `vec3(0, -1, -3)` |
| | focal_length | float | `0.4` |
| | image_plane_width | float | `0.6` |
| | image_plane_height | float | `0.6` |
| **Light** | position | vec3 | `vec3(3, 5, 2)` |
| | color | vec3 | `vec3(1.0, 1.0, 1.0)` |
| **Sphere** | center | vec3 | `vec3(0, 1, -4)` |
| | radius | float | `1.0f` |
| | color | vec3 | `vec3(0.0, 0.0, 1.0)` |
| **Triangle** | vertex_a | vec3 | `vec3(-1, 0, -7)` |
| | vertex_b | vec3 | `vec3(1, 0, -7)` |
| | vertex_c | vec3 | `vec3(0, 1, -5)` |
| | color | vec3 | `vec3(1.0, 0.0, 0.0)` |

---

## Anti-Aliasing & Recursion (Week 6)

```cpp
int maxDepth = 4;           // recursion depth limit
int rpp_NSquare = 4;        // 4×4 samples per pixel

for (int x = 0; x < image_width; x++) {
  for (int y = 0; y < image_height; y++) {
    vec3 accumulatedColor(0.0, 0.0, 0.0);
    
    for (int p = 0; p < rpp_NSquare; p++) {
      for (int q = 0; q < rpp_NSquare; q++) {
        float pOffset = (p + randomOffset()) / rpp_NSquare;
        float qOffset = (q + randomOffset()) / rpp_NSquare;
        
        ray r = cam.generateRay(x + pOffset, y + qOffset);
        accumulatedColor += computeRayColor(r, shapes, lights, maxDepth);
      }
    }
    
    vec3 pixelColor = accumulatedColor / (float)(rpp_NSquare * rpp_NSquare);
    fb.setPixelColor(x, y, pixelColor);
  }
}
```

---

[Home](index.md)
