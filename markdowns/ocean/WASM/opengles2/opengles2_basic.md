### opengl es 2 wasm template
```C++
#include <GLES2/gl2.h>
#include <SDL2/SDL.h>
#include <SDL2/SDL_video.h>

SDL_Window *window;

bool loop() {
  bool quit = false;
  SDL_Event event;
  while (SDL_PollEvent(&event)) {
    if (event.type == SDL_QUIT) {
      quit = true;
#ifdef __EMSCRIPTEN__
      emscripten_cancel_main_loop();
#endif
    }
  }
  return quit;
}

void wasmLoop() { loop(); }

int main(int argc, char *argv[]) {

  // SDL init

  window =
      SDL_CreateWindow("jesse window", SDL_WINDOWPOS_UNDEFINED,
                       SDL_WINDOWPOS_UNDEFINED, 800, 600, SDL_WINDOW_OPENGL);
  SDL_GLContext context = SDL_GL_CreateContext(window);


#ifdef __EMSCRIPTEN__
  emscripten_set_main_loop(wasmLoop, 0, 1);
#else
  bool quit = false;
  while (!quit) {
    quit = loop();
  }
#endif


  SDL_GL_DeleteContext(context);
  SDL_DestroyWindow(window);
  SDL_Quit();
  return 0;
}

```