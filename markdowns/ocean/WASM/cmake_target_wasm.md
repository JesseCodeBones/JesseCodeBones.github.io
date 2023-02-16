# generate wasm target using emsdk
* install emsdk: [GUIDE](https://emscripten.org/docs/getting_started/downloads.html)
* `emsdk install tot` `emsdk install latest`
* `emsdk activate tot` `emsdk activate latest`
* `source [emsdk dir]/emsdk_env.sh`
* `make dir build && cd build`
* `emcmake cmake .. -DCMAKE_BUILD_TYPE=Release`
* `emmake make -j2 asc-cov-instru`
