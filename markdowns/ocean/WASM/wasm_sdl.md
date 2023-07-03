### compile option:
`em++ main.cpp -s USE_SDL=2 -o index.html --emrun -s SINGLE_FILE`

### two rect
build:  
`em++ main.cpp -s USE_SDL=2 -o index.html --emrun -s SINGLE_FILE`
source code:  
```
#include <SDL2/SDL.h>
#include <math.h>
#ifdef __EMSCRIPTEN__
#include "emscripten/emscripten.h"
#endif

void loop() {
  SDL_Event event;
  while (SDL_PollEvent(&event)) {
    if (event.type == SDL_QUIT) {
#ifdef __EMSCRIPTEN__
      emscripten_cancel_main_loop();
#endif
    }
  }
}

int main(int argc, char *argv[]) {
  /* 初始化并创建窗口 */
  SDL_Init(SDL_INIT_EVERYTHING);
  SDL_Window *win = NULL;
  win = SDL_CreateWindow("SDL2", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED,
                         640, 400, SDL_WINDOW_RESIZABLE);

  /* 创建Renderer */
  SDL_Renderer *render = NULL;
  render = SDL_CreateRenderer(win, -1, SDL_RENDERER_ACCELERATED);

  /* 使用黑色清空窗口 */
  SDL_SetRenderDrawColor(render, 0, 0, 0, 0);
  SDL_RenderClear(render);

  /* 设置混合方式 */
  SDL_SetRenderDrawBlendMode(render, SDL_BLENDMODE_ADD);

  /* 设置颜色,画一个充填长方形 */
  SDL_SetRenderDrawColor(render, 0, 0xff, 0, 0xff);
  SDL_Rect rect; // 长方形结构体
  rect.x = 100;
  rect.y = 100;
  rect.w = 100;
  rect.h = 100;

  /* 画一个另一种颜色的充填长方形 */
  SDL_RenderFillRect(render, &rect);
  SDL_SetRenderDrawColor(render, 0, 0, 0xff, 0xff);
  rect.x = 150;
  rect.y = 150;
  SDL_RenderFillRect(render, &rect);

  /* 显示刷新 */
  SDL_RenderPresent(render);

  /* 等待退出 */
#ifdef __EMSCRIPTEN__
  emscripten_set_main_loop(loop, 0, 1);
#else
  SDL_Event e;
  while (1) {
    SDL_PollEvent(&e);
    if (e.type == SDL_QUIT) {
      break;
    }
  }
#endif
  /* 销毁renderer */
  SDL_DestroyRenderer(render);
  /* 销毁窗口 */
  SDL_DestroyWindow(win);
  /* 关闭SDL子系统 */
  SDL_Quit();

  return 0;
}

```