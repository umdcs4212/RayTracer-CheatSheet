# Week 2 — vec3 + Framebuffer + Export a PNG

By the end of Week 2, you should be able to run a program that generates a simple PNG image using your own `Framebuffer` and `vec3` classes.

---

## You are done if…

You have all of the following working:

- A Git repository for your ray tracer project
- A working `vec3.h` implementation (vector math)
- (OPTIONAL) A working `color.h` alias (`using color = vec3`)
- A `Framebuffer` class that stores pixel colors
- A standalone program `fbMain` that generates a PNG file (ex: `gradient.png`)
- Your CMake project builds successfully after adding new files and targets

---

## Files added in Week 2

These are the core files introduced this week:

```
vec3.h
color.h
Framebuffer.h
Framebuffer.cpp
fbMain.cpp
```

---

## What your CMake is doing this week

Week 2 introduces **two important CMake concepts**:

### 1) A library target: `cs4212-util`

Your Week 2 files are compiled into the `cs4212-util` library:

- `vec3.h`
- `color.h`
- `Framebuffer.cpp`
- `Framebuffer.h`

This is done inside the `add_library(cs4212-util ...)` block.

---

### 2) A new executable target: `fbMain`

A new executable is added:

- `fbMain.cpp`

This is done inside:

- `add_executable(fbMain fbMain.cpp)`

And it links against:

- `cs4212-util`
- `Boost::program_options`
- `PNG::PNG`
- `ZLIB::ZLIB`

This linking is required because `Framebuffer.cpp` uses **png++**, which depends on libpng + zlib.

---

## How your code might look like

The following are the code snippets that you might follow to get a working framebuffer. Note that, the codes and the design may vary from person to person and you can have a separate implementation that works fine (and better)!

### vec3.h

```cpp
#ifndef VEC3_H
#define VEC3_H

#include <cmath>
#include <iostream>

class vec3 {
  public:
    double e[3];

    vec3() : e{0,0,0} {}
    vec3(double e0, double e1, double e2) : e{e0, e1, e2} {}

    double x() const { return e[0]; }
    double y() const { return e[1]; }
    double z() const { return e[2]; }

    vec3 operator-() const { return vec3(-e[0], -e[1], -e[2]); }
    double operator[](int i) const { return e[i]; }
    double& operator[](int i) { return e[i]; }

    vec3& operator+=(const vec3& v) {
        e[0] += v.e[0];
        e[1] += v.e[1];
        e[2] += v.e[2];
        return *this;
    }

    vec3& operator*=(double t) {
        e[0] *= t;
        e[1] *= t;
        e[2] *= t;
        return *this;
    }

    vec3& operator/=(double t) {
        return *this *= 1/t;
    }

    double length() const {
        return std::sqrt(length_squared());
    }

    double length_squared() const {
        return e[0]*e[0] + e[1]*e[1] + e[2]*e[2];
    }
};

// point3 is just an alias for vec3, but useful for geometric clarity in the code.
using point3 = vec3;


// Vector Utility Functions

inline std::ostream& operator<<(std::ostream& out, const vec3& v) {
    return out << v.e[0] << ' ' << v.e[1] << ' ' << v.e[2];
}

inline vec3 operator+(const vec3& u, const vec3& v) {
    return vec3(u.e[0] + v.e[0], u.e[1] + v.e[1], u.e[2] + v.e[2]);
}

inline vec3 operator-(const vec3& u, const vec3& v) {
    return vec3(u.e[0] - v.e[0], u.e[1] - v.e[1], u.e[2] - v.e[2]);
}

inline vec3 operator*(const vec3& u, const vec3& v) {
    return vec3(u.e[0] * v.e[0], u.e[1] * v.e[1], u.e[2] * v.e[2]);
}

inline vec3 operator*(double t, const vec3& v) {
    return vec3(t*v.e[0], t*v.e[1], t*v.e[2]);
}

inline vec3 operator*(const vec3& v, double t) {
    return t * v;
}

inline vec3 operator/(const vec3& v, double t) {
    return (1/t) * v;
}

inline double dot(const vec3& u, const vec3& v) {
    return u.e[0] * v.e[0]
         + u.e[1] * v.e[1]
         + u.e[2] * v.e[2];
}

inline vec3 cross(const vec3& u, const vec3& v) {
    return vec3(u.e[1] * v.e[2] - u.e[2] * v.e[1],
                u.e[2] * v.e[0] - u.e[0] * v.e[2],
                u.e[0] * v.e[1] - u.e[1] * v.e[0]);
}

inline vec3 unit_vector(const vec3& v) {
    return v / v.length();
}

#endif
```

### color.h

```cpp
#ifndef COLOR_H
#define COLOR_H

#include "vec3.h"

#include <iostream>

using color = vec3;

inline void write_color(std::ostream& out, const color& pixel_color) {
    auto r = pixel_color.x();
    auto g = pixel_color.y();
    auto b = pixel_color.z();

    // Translate the [0,1] component values to the byte range [0,255].
    int rbyte = int(255.999 * r);
    int gbyte = int(255.999 * g);
    int bbyte = int(255.999 * b);

    // Write out the pixel color components.
    out << rbyte << ' ' << gbyte << ' ' << bbyte << '\n';
}

#endif
```

### Framebuffer.h

```cpp
#pragma once

#include <vector>
#include "vec3.h"
#include "color.h"

class Framebuffer {
    public:
    Framebuffer();
    Framebuffer(int width, int height);

    void clearToColor(const color& c);
    void clearToGradient(const color& c1, const color& c2);

    void setPixelColor(int i, int j, const color& c);
    void setPixelColor(int idx, const color& c);

    void exportToPNG(const std::string& filename);

    private:
    int width, height;
    std::vector<color> fbStorage;
};
```

### Framebuffer.cpp

```cpp
#include "Framebuffer.h"
#include "png++/png.hpp"

Framebuffer::Framebuffer() : width(100), height(100), fbStorage(width * height) {}

Framebuffer::Framebuffer(int width, int height) : width(width), height(height), fbStorage(width * height) {}

// Clear the framebuffer to a single color.
void Framebuffer::clearToColor(const color &c)
{
  for (auto idx = 0; idx < fbStorage.size(); idx++) {
    fbStorage[idx] = c;
  }
}

// Clear the framebuffer to a horizontal gradient between two colors, using linear interpolation.
void Framebuffer::clearToGradient(const color &c1, const color &c2)
{
  for (auto x = 0; x < width; x++) {
    for (auto y = 0; y < height; y++) {
      auto idx = y * width + x;
      auto t = double(y) / (height);

      color c = (1 - t) * c1 + t * c2;

      fbStorage[idx] = c;
    }
  }
}

// Set the color of a pixel at (i, j) in the framebuffer.
void Framebuffer::setPixelColor(int i, int j, const color &c)
{
  auto idx = j * width + i;
  fbStorage[idx] = c;
}

// Set the color of a pixel at the given index in the framebuffer.
void Framebuffer::setPixelColor(int idx, const color &c)
{
  fbStorage[idx] = c;
}

// Export the framebuffer to a PNG file with the given filename.
void Framebuffer::exportToPNG(const std::string &filename)
{
  png::image<png::rgb_pixel> imData(width, height);

  for (int j = 0; j < height; ++j) {
    for (int i = 0; i < width; ++i) {
      vec3 color = fbStorage[j * width + i];

      png::byte r = static_cast<png::byte>(color.x() * 255.0);
      png::byte g = static_cast<png::byte>(color.y() * 255.0);
      png::byte b = static_cast<png::byte>(color.z() * 255.0);

      imData[j][i] = png::rgb_pixel(r, g, b);
    }
  }

  imData.write(filename);
}
```

### fbMain.cpp

```cpp
#include <iostream>
#include "Framebuffer.h"

int main(int argc, char *argv[])
{
  Framebuffer fb(800, 600);

  color red(1.0, 0.0, 0.0);
  color blue(0.0, 0.0, 1.0);

  fb.clearToGradient(red, blue);

  fb.exportToPNG("gradient.png");

  return 0;
}
```

### CMakeLists.txt

```txt
add_library (cs4212-util
  ArgumentParsing.cpp ArgumentParsing.h
  handleGraphicsArgs.cpp handleGraphicsArgs.h
  model_obj.cpp model_obj.h
  vec3.h color.h
  Framebuffer.cpp Framebuffer.h
)
target_compile_definitions(cs4212-util PUBLIC HAS_GLM)
target_link_libraries(cs4212-util PRIVATE Boost::program_options)
target_link_libraries(cs4212-util PUBLIC glm::glm)

add_executable(fbMain
  fbMain.cpp
)

target_link_libraries(fbMain PRIVATE cs4212-util)
target_link_libraries(fbMain PRIVATE ${Boost_PROGRAM_OPTIONS_LIBRARIES})
target_link_libraries(fbMain PRIVATE PNG::PNG)
target_link_libraries(fbMain PRIVATE ZLIB::ZLIB)


```


## What your program should produce

When you run:

```bash
./src/fbMain
```
from your buildVCPkg directory

You should get a PNG file in your build directory:

```
gradient.png
```

The image should show a clear gradient (red -> blue).

---

## Quick build / run commands

From the root of your project:

```bash
cd buildVCPkg
cmake ..
cmake --build .
./src/fbMain
```


## Self-check: Debug checklist

If your output image is wrong or missing, check these:

### “My PNG file is not created”
- Are you running `fbMain` from the correct folder?
- Are you sure the program is running successfully (no crash)?
- Try printing a message right before `exportToPNG(...)`.

---

### “My PNG is black”
Common causes:
- you never changed the framebuffer storage values
- your colors are all `(0,0,0)`
- your color values are outside the expected range `[0,1]`

---

### “Build fails with PNG / ZLIB errors”
That usually means:
- `png++` headers aren’t being found
- libpng or zlib are not linked correctly
- your system is missing dependencies

If it fails, it usually means the libraries aren’t installed on the machine.

---

### “I added a file but nothing changed”
That usually means:
- you created the `.cpp` file but forgot to add it to the correct CMake target
- you modified CMake but didn’t re-run configure



---

[Home](index.md) | [Week 3](week3.md)
