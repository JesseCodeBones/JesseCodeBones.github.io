# generate wasm target using emsdk
* install emsdk: [GUIDE](https://emscripten.org/docs/getting_started/downloads.html)
* `emsdk install tot` `emsdk install latest`
* `emsdk activate tot` `emsdk activate latest`
* `source [emsdk dir]/emsdk_env.sh`
* `make dir build && cd build`
* `emcmake cmake .. -DCMAKE_BUILD_TYPE=Release`
* `emmake make -j2 asc-cov-instru`

## install wasi sdk
```
# install wasi-sdk
ARG WASI_SDK_VER=21
RUN wget -c --progress=dot:giga https://github.com/WebAssembly/wasi-sdk/releases/download/wasi-sdk-${WASI_SDK_VER}/wasi-sdk-${WASI_SDK_VER}.0-linux.tar.gz -P /opt \
  && tar xf /opt/wasi-sdk-${WASI_SDK_VER}.0-linux.tar.gz -C /opt \
  && ln -sf /opt/wasi-sdk-${WASI_SDK_VER}.0 /opt/wasi-sdk \
  && rm /opt/wasi-sdk-${WASI_SDK_VER}.0-linux.tar.gz
```
