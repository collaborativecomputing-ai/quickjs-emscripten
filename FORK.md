# Fork Changes

This is a fork of [justjake/quickjs-emscripten](https://github.com/justjake/quickjs-emscripten) with changes for Metro/Expo compatibility in [Atelier](https://github.com/collaborativecomputing-ai/atelier).

## Changes

### 1. Remove Node.js environment from singlefile-cjs target

**File:** `scripts/prepareVariants.ts`

**Change:** Removed `EmscriptenEnvironment.node` from the `singlefile-cjs` target's `emscriptenEnvironment` list.

**Motivation:** Metro (Expo's bundler) processes all code paths at bundle time, even dead ones. The CJS singlefile variant included `ENVIRONMENT=web,worker,node` which caused Emscripten to generate `require("fs")` and `require("path")` calls in the JS glue — even though the WASM is embedded as base64 and no file I/O occurs at runtime. Metro can't resolve `fs`/`path` in a browser bundle, causing `Requiring unknown module` errors. Removing `node` from the environment list tells Emscripten to omit the Node.js code path entirely.

### 2. Add `-DEMSCRIPTEN` to QuickJS build defines

**File:** `templates/Variant.mk`

**Change:** Added `QUICKJS_DEFINES+=-DEMSCRIPTEN` after the QuickJS object list.

**Motivation:** QuickJS's `quickjs.c` has platform-specific code guarded by `#ifdef EMSCRIPTEN` (e.g., `malloc_usable_size` which doesn't exist in Emscripten's libc). However, newer Emscripten versions (5.x) define `__EMSCRIPTEN__` (with double underscores) but not the bare `EMSCRIPTEN`. Without this define, QuickJS falls into the `#else` branch and calls `malloc_usable_size`, causing a compile error. Adding `-DEMSCRIPTEN` explicitly ensures the correct Emscripten code path is taken.

### 3. Update default Emscripten SDK version to 5.0.4

**File:** `scripts/prepareVariants.ts`

**Change:** Updated `DEFAULT_EMSCRIPTEN_VERSION` from `"5.0.1"` to `"5.0.4"`.

**Motivation:** Matches our local Emscripten SDK installation. The build script (`scripts/emcc.sh`) checks the local `emcc` version against the expected version and falls back to Docker if they don't match. Updating this avoids requiring Docker for builds.
