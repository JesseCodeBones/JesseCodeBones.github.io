## std::bind and std::ref
```C++
void changeValue(int &a) { a++; }

int main() {
  int value = 0;
  auto f1 = std::bind(changeValue, value); // this one wil not change value value
  f1();
  auto f2 = std::bind(changeValue, std::ref(value)); // this one will move reference to function
  f2();
  std::cout << value << std::endl;
}
```