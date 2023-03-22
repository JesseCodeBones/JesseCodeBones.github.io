# C++内存操作技巧
### shared_ptr循环引用
当多个对象之间存在循环引用（circular reference）时，使用 shared_ptr 可能会导致内存泄漏，因为每个对象都持有另一个对象的 shared_ptr 导致计数器无法降到0，从而无法自动释放内存。  
例子：

```C++
class Node {
public:
    std::shared_ptr<Node> next;
    ~Node() { cout << "Node destroyed." << endl; }
};

int main()
{
    std::shared_ptr<Node> node1(new Node());
    std::shared_ptr<Node> node2(new Node());
    node1->next = node2;
    node2->next = node1; // 循环引用
    return 0;
}
```

改进后的代码：  
```C++
#include <iostream>
#include <memory>
class Node {
public:
    std::weak_ptr<Node> next; // <- 避免循环引用
    ~Node() { std::cout << "Node destroyed." << std::endl; }
};

int main()
{
    std::shared_ptr<Node> node1(new Node());
    std::shared_ptr<Node> node2(new Node());
    node1->next = node2;
    node2->next = node1; // 循环引用
    std::cout << "before destroy\n";
    return 0;
}```