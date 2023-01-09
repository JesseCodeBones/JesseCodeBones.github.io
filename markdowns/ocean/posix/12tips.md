# useful API from posix

### qsort() sort array
```C++
int testArray[] = {-10, -15, 3, 2, 8, 9};
  qsort(testArray, sizeof(testArray) / sizeof(testArray[0]), sizeof(int),
        [](const void *a, const void *b) { return *(int *)a - *(int *)b; });
  for (int i = 0; i < 6; ++i) {
    std::cout << testArray[i] << std::endl;
  }
```