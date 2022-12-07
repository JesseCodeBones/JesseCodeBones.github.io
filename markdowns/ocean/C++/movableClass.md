# Movable Class
```C++
class MovableClass {
public:
  MovableClass() { std::cout << "default construct \n"; }
  MovableClass(const MovableClass &src) { std::cout << "copy construct \n"; }
  MovableClass(MovableClass &&src) { std::cout << "move construct \n"; }
  MovableClass &operator=(const MovableClass &src) {
    std::cout << "copy operator \n";
    return *this;
  }
  MovableClass &operator=(MovableClass &&src) {
    std::cout << "move operator \n";
    return *this;
  }
  friend std::ostream &operator<<(std::ostream &os, const MovableClass &dt) {
    os << "hello from jesse";
    return os;
  };
};
```