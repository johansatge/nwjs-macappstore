
# Compiling a custom binary

Follow this guide if you want to compile the MAS-compatible binary by yourself.

You will need to download the source code and apply various patches on it.

This guide is based on @alexeyst's [node-webkit-macappstore](https://github.com/alexeyst/node-webkit-macappstore/blob/master/node-webkit-mas.diff) repository.

## Get the source

### Get the Chromium depot_tools

[Instructions available here](http://www.chromium.org/developers/how-tos/install-depot-tools)

### Get nw.js

Create a directory wherever you want, the following instructions assume it is named `nwjs-source`.

Create a `.gclient` file with the following content:

```
solutions = [
   { "name"        : "src",
     "url"         : "https://github.com/nwjs/chromium.src.git@origin/nw12",
     "deps_file"   : "DEPS",
     "managed"     : True,
     "custom_deps" : {
       "src/third_party/WebKit/LayoutTests": None,
       "src/chrome_frame/tools/test/reference_build/chrome": None,
       "src/chrome_frame/tools/test/reference_build/chrome_win": None,
       "src/chrome/tools/test/reference_build/chrome": None,
       "src/chrome/tools/test/reference_build/chrome_linux": None,
       "src/chrome/tools/test/reference_build/chrome_mac": None,
       "src/chrome/tools/test/reference_build/chrome_win": None,
     },
     "safesync_url": "",
   },
]
```

Download the code by running:

```bash
cd nwjs-source
gclient sync --no-history
```

*Downloaded code can use up to 4GB.*

### Get the patches

```bash
# Assumes cwd is nwjs-source
curl https://raw.githubusercontent.com/alexeyst/node-webkit-macappstore/master/node-webkit-mas.diff > node-webkit-mas.diff
curl https://raw.githubusercontent.com/alexeyst/node-webkit-macappstore/master/webkit.diff > webkit.diff
```

## Patch NW.js

```bash
# Assumes cwd is nwjs-source
cd src
git apply ../node-webkit-mas.diff
```

## Patch WebKit

```bash
# Assumes cwd is nwjs-source
cd src/third_party/WebKit
git init
git apply ../../../webkit.diff
```

*Project dependencies are located in the `third_party` directory, and are ignored, so we init an empty Git repo to apply the patch.*

# Build

```bash
# Assumes cwd is nwjs-source
export GYP_GENERATORS='ninja'
./build/gyp_chromium content/content.gyp --no-circular-check
ninja -C out/Release nw -j4
```

*/!\ The process can use up to 30GB.*

Some warnings may appear during the compilation.

When it is done, you can get test binary: `nwjs-source/src/out/Release/nwjs.app`.
