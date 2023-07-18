### vertex shader & fragment shader

### emscripten loop回调函数
`callUserCallback`  

### 创建shader的运行函数
```C++
  char vShaderStr[] = "attribute vec4 vPosition;\n"
                      "void main()\n"
                      "{\n"
                      "gl_Position = vPosition; \n"
                      "}\n";
  char fShaderStr[] = "precision mediump float;\n"
                      "void main()\n"
                      "{\n"
                      " gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0); \n"
                      "}\n";
```

### 创建loadShader函数
```C++
GLuint loadShader(GLenum type, const char *shaderSrc) noexcept {
  GLuint shader;
  GLint compiled;

  shader = glCreateShader(type);
  if (shader == 0U) {
    return 0U;
  }
  glShaderSource(shader, 1, &shaderSrc, nullptr);
  glCompileShader(shader);
  glGetShaderiv(shader, GL_COMPILE_STATUS, &compiled);
  if (!compiled) {
    // 编译失败，打印提示错误信息
    GLint infoLen = 0;
    glGetShaderiv(shader, GL_INFO_LOG_LENGTH, &infoLen);
    if (infoLen > 0) {
      char *infoLog = (char *)malloc(sizeof(char) * infoLen);
      glGetShaderInfoLog(shader, infoLen, nullptr, infoLog);
      printf("%s\n", shaderSrc);
      printf("%s\n", infoLog);
      free(infoLog);
    }
    glDeleteShader(shader);
    return shader;
  }
  return shader;
}
```
需要注意的是这里的printf，在emscripten的工作页面是可以直接打印到页面下面的，所以非常有用，尽量攫取opengl的错误信息然后打印到console上。  


### 创建两个Shader, 一个vertex shader一个fragment shader
```C++
  GLuint vertexShader = loadShader(GL_VERTEX_SHADER, vShaderStr);
  GLuint fragmentShader = loadShader(GL_FRAGMENT_SHADER, fShaderStr);
```

### 创建program，attach shader，链接program， use program
```C++
// create program and attach shader
  GLuint program = glCreateProgram();
  if (program == 0) {
    return 0;
  }
  glAttachShader(program, vertexShader);
  glAttachShader(program, fragmentShader);
  std::cout << "program:" << program << ",vertex:" << vertexShader
            << ",fragment:" << fragmentShader << std::endl;

  // bind vertex attribute vPosition
  // glBindAttribLocation(program, 0, "vPosition");

  // link program and check error
  glLinkProgram(program);
  GLint linked;
  glGetProgramiv(program, GL_LINK_STATUS, &linked);
  if (!linked) {
    SDL_LogError(SDL_LOG_CATEGORY_ERROR, "program link error!");
    glDeleteProgram(program);
    std::cout << "cannot link to program\n";
    return 1;
  }

  userData.program = program;
  glUseProgram(userData.program);
```

### 确定定点位置
```C++
  // 确定顶点位置
  GLfloat vVertices[] = {0.0f, 0.5f, 0.0f,  -0.5f, -0.5f,
                         0.0f, 0.5f, -0.5f, 0.0f};
  glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, 0, vVertices);
  glEnableVertexAttribArray(0);
```

### 编写循环渲染程序
```C++
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
  glClear(GL_COLOR_BUFFER_BIT);
  glDrawArrays(GL_TRIANGLES, 0, 3);
  // 刷新window
  SDL_GL_SwapWindow(window); // swap back buffer and front buffer, to display
                             // the ready buffer
  return quit;
}
```
### 为emscripten包装一层loop方法
```C++
void wasmLoop() { loop(); }
```

### 循环模板写法：
```C++
#ifdef __EMSCRIPTEN__
  emscripten_set_main_loop(wasmLoop, 0, 1);
#else
  bool quit = false;
  while (!quit) {
    quit = loop();
  }
#endif
```

### 推出时清理
```C++
  SDL_GL_DeleteContext(context);
  SDL_DestroyWindow(window);
  SDL_Quit();
```
