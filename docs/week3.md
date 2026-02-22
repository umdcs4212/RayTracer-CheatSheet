# Week 3 — Rays + Camera + Ray-Color Image

During this week, we conceptualize the idea of a **ray** and a **basic camera**. These components will help us in the coming weeks to render images of 3D scenes.

By the end of Week 3, you should be able to:
- create rays in your code
- use a basic camera to generate rays
- use the **ray direction** to color your pixels in your framebuffer
- export a PNG image from your `Framebuffer` object

The goal is to ensure you can write PNG images, understand what a ray is, and generate rays from a camera so you’re ready to start real ray tracing computations next week.

---

## You are done if…

You have all of the following working:

- A working `ray` type (`ray.h`) with origin, direction, and `at(t)`
- A `Camera` base class with a pure virtual `generateRay(i, j)`
- A `PerspectiveCamera` class that implements `generateRay(i, j)`
- A driver program (`fbMain.cpp`) that:
  - loops over pixels
  - generates a ray per pixel
  - colors the pixel based on ray direction
  - exports a PNG image (ex: `defaultCamRayColors.png`)

---

## Files added / modified in Week 3

Core files introduced this week:

```
ray.h
Camera.h
Camera.cpp
PerspectiveCamera.h
PerspectiveCamera.cpp
fbMain.cpp   (updated to use rays + camera)
```

---

## What your output should look like

When you run your Week 3 program, you should produce an image like:

```
defaultCamRayColors.png
```

The image should look something like this - 

![defaultCamRayColors](../images/defaultCamRayColors.png)

---

## CMake reminder (most common issue)

If you added new `.cpp` files (like `Camera.cpp` and `PerspectiveCamera.cpp`), make sure they are included in the correct CMake target (usually your utility library).

If a `.cpp` file is not listed in the target, your code might compile partially but fail to link, or your changes won’t be included.

---

## How your code might look like

The following are example snippets matching this week’s structure. Your design may vary — that’s okay — as long as your code meets the Week 3 checklist.

### ray.h

```cpp
#ifndef RAY_H
#define RAY_H

#include "vec3.h"

class ray {
  public:
    ray() {}

    ray(const point3& origin, const vec3& direction) : orig(origin), dir(direction) {}

    const point3& origin() const  { return orig; }
    const vec3& direction() const { return dir; }

    point3 at(double t) const {
        return orig + t*dir;
    }

  private:
    point3 orig;
    vec3 dir;
};

#endif
```

---

### Camera.h

```cpp
#pragma once

#include "ray.h"

class Camera
{
public:
  Camera();
  Camera(int pixel_nx, int pixel_ny);
  Camera(vec3 position, vec3 viewDir, vec3 upDir, float focal_length, float image_plane_width, float image_plane_height, int pixel_nx, int pixel_ny);

  virtual ray generateRay(int i, int j) = 0;

protected:
  // position of the camera
  vec3 pos;

  // the camera's basis vectors
  vec3 U, V, W;

  // focal length
  float focalLength;

  // imageplane dimensions
  float imagePlaneWidth, imagePlaneHeight;

  // number of pixels in x and y direction
  int nx, ny;
};
```

---

### Camera.cpp

```cpp
#include "Camera.h"

Camera::Camera()
: pos(vec3(0, 0, 0)),
   U(vec3(1, 0, 0)),
   V(vec3(0, 1, 0)),
   W(vec3(0, 0, 1)),
   focalLength(1.0),
   imagePlaneWidth(0.25),
   imagePlaneHeight(0.25),
   nx(100),
   ny(100)
{
}

Camera::Camera(int pixel_nx, int pixel_ny)
: pos(vec3(0, 0, 0)),
   U(vec3(1, 0, 0)),
   V(vec3(0, 1, 0)),
   W(vec3(0, 0, 1)),
   focalLength(1.0),
   imagePlaneWidth(0.25),
   imagePlaneHeight(0.25),
   nx(pixel_nx),
   ny(pixel_ny)
{
}

Camera::Camera(vec3 position, vec3 viewDir, vec3 upDir, float focal_length, float image_plane_width, float image_plane_height, int pixel_nx, int pixel_ny)
: pos(position),
   focalLength(focal_length),
   imagePlaneWidth(image_plane_width),
   imagePlaneHeight(image_plane_height),
   nx(pixel_nx),
   ny(pixel_ny)
{
    W = -unit_vector(viewDir);
    
    vec3 t = upDir;

    // If the up direction is parallel to the view direction, we need to choose a different up vector to avoid a degenerate camera basis.
    if (std::abs(dot(unit_vector(t), W)) > 0.999f) {
        t = vec3(0, 0, 1); // Choose a different up vector
    }

    U = unit_vector(cross(t, W));
    V = cross(W, U);
}
```

---

### PerspectiveCamera.h / PerspectiveCamera.cpp

```cpp
#pragma once

#include "Camera.h"

class PerspectiveCamera : public Camera
{
public:
    PerspectiveCamera();
    PerspectiveCamera(int pixel_nx, int pixel_ny);
    PerspectiveCamera(vec3 origin, vec3 viewDir, float focal_length, float image_plane_width, float image_plane_height, int pixel_nx, int pixel_ny);

    ray generateRay(int i, int j) override;

private:
    float left, right, top, bottom;
};
```

```cpp
#include "PerspectiveCamera.h"

PerspectiveCamera::PerspectiveCamera()
: Camera()
{
    left = -imagePlaneWidth / 2.0;
    right = imagePlaneWidth / 2.0;
    bottom = -imagePlaneHeight / 2.0;
    top = imagePlaneHeight / 2.0;
}

PerspectiveCamera::PerspectiveCamera(int pixel_nx, int pixel_ny)
: Camera(pixel_nx, pixel_ny)
{
    left = -imagePlaneWidth / 2.0;
    right = imagePlaneWidth / 2.0;
    bottom = -imagePlaneHeight / 2.0;
    top = imagePlaneHeight / 2.0;
} 
  
PerspectiveCamera::PerspectiveCamera(vec3 origin, vec3 viewDir, float focal_length, float image_plane_width, float image_plane_height, int pixel_nx, int pixel_ny)
: Camera(origin, viewDir, vec3(0, 1, 0), focal_length, image_plane_width, image_plane_height, pixel_nx, pixel_ny)
{
    left = -imagePlaneWidth / 2.0;
    right = imagePlaneWidth / 2.0;
    bottom = -imagePlaneHeight / 2.0;
    top = imagePlaneHeight / 2.0;
}

ray PerspectiveCamera::generateRay(int i, int j)
{
    float u = left + (right - left) * (i + 0.5) / (float)nx;
    float v = bottom + (top - bottom) * (j + 0.5) / (float)ny;
    vec3 rayDir = -focalLength * W + u * U + v * V;

    return ray(pos, rayDir);
}
```

---

### fbMain.cpp

The idea: generate one ray per pixel, then map ray direction to a visible color.

```cpp
#include <iostream>
#include "Framebuffer.h"
#include "PerspectiveCamera.h"
#include "ray.h"

vec3 computeRayColor(const ray& r) {
  vec3 unit_direction = unit_vector(r.direction());

  auto a = 0.5 * (unit_direction.y() + 1.0);

  return (1.0 - a) * vec3(1.0, 1.0, 1.0) + a * vec3(0.5, 0.7, 1.0);
}

int main(int argc, char *argv[])
{
  Framebuffer fb(800, 800);

  PerspectiveCamera cam;

  for (int x = 0; x < 800; x++) {
    for (int y = 0; y < 800; y++) {
      ray r;

      r = cam.generateRay(x, y);

      vec3 pixelColor = computeRayColor(r);

      fb.setPixelColor(x, y, pixelColor);
    }
  }

  fb.exportToPNG("defaultCamRayColors.png");

  return 0;
}
```

---

## Self-check: Debug checklist

### “My image is a single solid color”
- Your camera might be generating the same ray for every pixel.
- Double-check you are using `(i, j)` inside `generateRay(i, j)`.

### “My output is black or mostly black”
- Your colors might not be in `[0, 1]`.
- Ray-direction visualization should map values into `[0,1]` before writing.

### “It builds but I get link errors for camera”
- You likely forgot to add `Camera.cpp` / `PerspectiveCamera.cpp` to the CMake target.

### “The image is upside down”
- This can happen depending on pixel loop direction and PNG row indexing.
- Not a problem as long as you understand why.

---


[Week2](week2.md) | [Home](index.md) | [Week4](week4.md)
