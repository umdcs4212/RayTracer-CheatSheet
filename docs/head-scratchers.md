# Head-scratchers

The small setup problems that eat an afternoon. Fix these first so the weekly
checkpoints are about ray tracing and not about your toolchain.

## Contents

- [1. Cloning and setting up your own repository](#1-cloning-and-setting-up-your-own-repository)
- [2. How to actually build and run a file](#2-how-to-actually-build-and-run-a-file)
- [3. Using your own subdirectory instead of `src`](#3-using-your-own-subdirectory-instead-of-src)
- [4. Catch2 tests](#4-catch2-tests)
- [5. Adding and running your own test file](#5-adding-and-running-your-own-test-file)

---

## 1. Cloning and setting up your own repository

Visit the following to have a basic idea:

[Repository Setup](https://ananan.notion.site/CS4212-5212-GitHub-Repository-Setup-3d34fa99eebf80338197c27fb8eab287)

[Back to top](#head-scratchers)

---

## 2. How to actually build and run a file

Once CMake has configured the project, you build from your build directory and
run the executable it produced.

### macOS / Linux

```bash
cd buildVCPkg
cmake ..

# every time after that
cmake --build .

# run an executable (built next to its source subdirectory)
./src/fbMain
```

The build drops each executable directly in the subdirectory that matches its
CMake target, e.g. `buildVCPkg/src/fbMain`.

### Windows

The commands are the same up to `cmake --build .`, but the layout of the output
is different. On Windows (Visual Studio / MSBuild generator) CMake builds a
**configuration** (`Debug` by default) and puts each executable **inside a
`Debug` folder within that target's subdirectory**:

```powershell
cd buildVCPkg
cmake ..
cmake --build .

# note the extra Debug\ segment, and the .exe extension
.\src\Debug\fbMain.exe
```

So the same program that lives at `buildVCPkg/src/fbMain` on macOS/Linux lives at
`buildVCPkg\src\Debug\fbMain.exe` on Windows. If you build the Release
configuration (`cmake --build . --config Release`) swap `Debug` for `Release`.

| | macOS / Linux | Windows |
|---|---|---|
| Build command | `cmake --build .` | `cmake --build .` |
| Executable path | `src/fbMain` | `src\Debug\fbMain.exe` |
| Per-config folder | none | `Debug\` or `Release\` |

[Back to top](#head-scratchers)

---

## 3. Using your own subdirectory instead of `src`

If `src` is getting crowded, you can keep your own files in a separate
subdirectory (e.g. `renderlib/`) instead of dumping everything into `src`. This is
just a CMake change — nothing about how you write your code changes.

**Steps:**

1. Create the new folder at the project root, next to `src`:

   ```
   your-project/
   ├── CMakeLists.txt
   ├── src/
   └── renderlib/          <- your new folder
       ├── CMakeLists.txt
       ├── vec3.h
       └── mainCode.cpp
   ```

2. Add a `CMakeLists.txt` inside `renderlib/` that declares your targets, the
   same way `src/CMakeLists.txt` does:

   ```cmake
   add_library(renderlib
     vec3.h
   )

   add_executable(mainCode
     mainCode.cpp # if you have an executable, list its .cpp file(s) here
   )

   target_link_libraries(mainCode PRIVATE renderlib)
   ```

   As you add more files later (e.g. a `Framebuffer` that needs PNG/ZLIB),
   link them the same way `src/CMakeLists.txt` already does — nothing about
   this pattern changes as the project grows.

3. Tell the **top-level** `CMakeLists.txt` that this folder exists, next to
   the existing `add_subdirectory(src)` line:

   ```cmake
   add_subdirectory(src)
   add_subdirectory(renderlib)
   ```

4. Re-run configure (not just build), since you changed the CMake project
   structure:

   ```bash
   cd buildVCPkg
   cmake ..
   cmake --build .
   ```

5. Your executables now show up under the new folder name instead of `src`:

   | | macOS / Linux | Windows |
   |---|---|---|
   | Executable path | `renderlib/mainCode` | `renderlib\Debug\mainCode.exe` |

   The `Debug\` folder rule from section 2 still applies — it's per target,
   not tied to the `src` name specifically.

**Gotchas:**
- Forgetting `add_subdirectory(renderlib)` in the top-level `CMakeLists.txt` — your
  new folder is silently ignored by CMake.
- Target names must be unique across the whole project — if `src` already has
  a target called `mainCode`, pick a different name for the one in `renderlib`.
- If your new files `#include` headers that are still in `src/` (because you
  only moved some of your files to `renderlib`), you'll need
  `target_include_directories` or `target_link_libraries` against the library
  that owns those headers.

[Back to top](#head-scratchers)

---

## 4. Catch2 tests

Catch2 testing tutorial: https://github.com/catchorg/Catch2/blob/devel/docs/tutorial.md#top

[Back to top](#head-scratchers)

---

## 5. Adding and running your own test file

Say you wrote a new test file `utests/utest_Vec3.cpp`. Two things have to
happen: CMake has to know it exists, and you have to re-configure so the new
test executable gets built and registered with CTest.

### Register the file in `utests/CMakeLists.txt`

`utests/CMakeLists.txt` builds one executable per name listed in the `UTESTS`
variable, then calls `catch_discover_tests()` on each. So you only add the new
file's name (without the `.cpp`) to that list:

```cmake
set(UTESTS
  utest_Success
  utest_Failure
  utest_Vec3)          # <- your new test file: utests/utest_Vec3.cpp
```

The `foreach` below it does the rest — `add_executable`, links
`Catch2::Catch2WithMain` and `cs4212-util`, and registers the individual
`TEST_CASE`s with CTest. If your test needs another library (e.g. your
`renderlib` from section 3), add it to the `target_link_libraries` call in the
loop.

### Re-configure and build

Because you edited a `CMakeLists.txt`, run configure again — a plain
`cmake --build .` is not enough:

```bash
cd buildVCPkg
cmake ..                 # picks up the new UTESTS entry
cmake --build .          # builds utest_Vec3 (and anything else stale)
```

### Run the tests

CTest is driven from the build directory. `catch_discover_tests` registers each
`TEST_CASE` as its own CTest entry, so you can run everything or pick one case.

```bash
cd buildVCPkg

# run every registered test
ctest
```

OR

```bash
cd buildVCPkg

# same thing via the build system's "test" target
cmake --build . --target test
```

OR

```bash
cd buildVCPkg

# run a single test case by its Catch2 name (regex)
ctest -R "Simple Vec3 addition test"
```

You can also skip CTest and run a test executable directly — this gives you
Catch2's own CLI (tag filters, `--list-tests`, `--success`, etc.):

```bash
# macOS / Linux
./utests/utest_Vec3                      # run all TEST_CASEs in the file
./utests/utest_Vec3 "Simple Vec3 addition test"  # run one by name

# Windows (note the Debug\ segment from section 2)
.\utests\Debug\utest_Vec3.exe
```

**Gotchas:**
- Ran `cmake --build .` and the new test isn't found — you forgot `cmake ..`
  after editing `utests/CMakeLists.txt`.
- `ctest` reports "No tests were found" — run it from inside `buildVCPkg`, not
  the project root.
- New `.cpp` compiles but no `TEST_CASE` runs — make sure it includes
  `<catch2/catch_test_macros.hpp>` and that you link `Catch2WithMain` (the
  `foreach` already does this; only a concern if you added the target by hand).

[Back to top](#head-scratchers)

---

[Home](index.md) | [Week 2](week2.md)
