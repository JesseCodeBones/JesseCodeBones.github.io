# C++ exception
### 常规exception 处理
```C++
{}
catch (...) {
    auto expr = std::current_exception();
    if (buffer != NULL) {
      free(buffer);
    }
    if (expr) {
      try {
        std::rethrow_exception(expr);
      } catch (const std::exception &e) {
        std::cerr << "Error happened during instrument - " << e.what()
                  << std::endl;
      }
    } else {
      std::cerr << "Error happened during instrument\n";
    }
  }
```
