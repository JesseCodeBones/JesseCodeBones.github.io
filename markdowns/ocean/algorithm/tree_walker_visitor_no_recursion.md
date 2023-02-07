## demo
```C++
#include <iostream>
#include <stack>
#include <vector>
// Define a tree node
struct Node {
  int data;
  std::vector<Node *> children;
};

// Define a visitor class
class Visitor {
public:
  virtual void visit(Node *node) = 0;
};

// Define a walker class
class Walker {
public:
  virtual void walk(Node *root, Visitor *visitor) = 0;
};

// Define a pre-order walker
class PreOrderWalker : public Walker {
private:
public:
  virtual void walk(Node *root, Visitor *visitor) { // walk tree without recursion
    std::stack<Node *> stack;
    stack.push(root);
    while (!stack.empty()) {
      auto node = stack.top();
      stack.pop();
      for (auto child : node->children) {
        stack.push(child);
      }
      visitor->visit(node);
    }
  }
};

// Define a visitor that prints each node
class PrintVisitor : public Visitor {
public:
  virtual void visit(Node *node) { std::cout << node->data << " "; }
};

int main() {
  Node root;
  root.data = 1;
  root.children.push_back(new Node{2});
  root.children.push_back(new Node{3});
  root.children[0]->children.push_back(new Node{4});
  root.children[0]->children.push_back(new Node{5});

  Walker *walker = new PreOrderWalker();
  Visitor *visitor = new PrintVisitor();
  walker->walk(&root, visitor);

  return 0;
}
```