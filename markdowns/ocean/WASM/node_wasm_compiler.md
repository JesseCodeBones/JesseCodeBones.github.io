# NodeJS WASM compiler

## Introduce
NodeJs WASM Compiler contains `Liftoff` and `Turbofan`  

## V8 project compile and run 

* prepare tool `git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git`
* add to path `export PATH=/home/jesse/workspace/depot_tools:$PATH`
* install dependencies `./build/install-build-deps.sh --no-chromeos-fonts`
* build debug version `alias gm=/home/jesse/workspace/v8/v8/tools/dev/gm.py && gm x64.debug`
* ninja build `ninja -C out/x64.debug`
* run test `tools/run-tests.py --outdir /home/jesse/workspace/v8/v8/out/x64.debug wasm-spec-tests`
* run unit test `./unittests --gtest_filter=HeapObjectHeaderDeathTest*`
* run spec test `./out/x64.debug/d8 --test /home/jesse/workspace/v8/v8/test/wasm-spec-tests/tests/const.js --random-seed=44050501 --nohard-abort --verify-heap --enable-slow-asserts --testing-d8-test-runner`

## flow

### uml: class diagram
![Class Diagram](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/JesseCodeBones/JesseCodeBones.github.io/main/markdowns/ocean/uml/WASM/v8_wasm_compiler.puml)
