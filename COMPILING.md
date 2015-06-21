
# Compiling a custom binary

Follow this guide if you want to compile the MAS-compatible binary by yourself.

You will need to download the source code and apply various patches on it.

# Get the source

## Get the Chromium depot_tools

[Instructions available here](http://www.chromium.org/developers/how-tos/install-depot-tools)

## Get nw.js

Create a directory wherever you want, the following instructions assume it is `nwjs-source`.

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

# patch nwjs

cd src

git apply node-webkit-mas.diff

# patch webkit

cd src/third_party/WebKit

git init

git apply webkit.diff

# build

export GYP_GENERATORS='ninja'

./build/gyp_chromium content/content.gyp --no-circular-check

ninja -C out/Release nw -j4
