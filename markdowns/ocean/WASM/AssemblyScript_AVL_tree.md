# AVL tree for AssemblyScript
## Code
```TypeScript
import { logi, log } from "./env";
export function main(): void {
    let tree = new AVLTree();
    tree.insert(10);
    tree.insert(5);
    tree.insert(15);
    tree.insert(3);
    tree.insert(7);
    tree.insert(13);
    tree.insert(17);
    log(tree.toString());
    let root = tree.root;
    if (root != null) {
        logi(root.value);
    }
}

class AVLNode {
    value: i32;
    left: AVLNode | null;
    right: AVLNode | null;
    balance: i32;
    parent: AVLNode | null;

    constructor(value: i32) {
        this.value = value;
        this.left = null;
        this.right = null;
        this.balance = 0;
        this.parent = null;
    }
}


class AVLTree {
    root: AVLNode | null;

    insert(value: i32): void {
        let newNode = new AVLNode(value);
        this.root = this._insert(this.root, newNode);

        this._rebalance(newNode);
    }

    private _insert(node: AVLNode | null, newNode: AVLNode): AVLNode {
        if (node == null) {
            return newNode;
        } else if (newNode.value < node.value) {
            node.left = this._insert(node.left, newNode);
            let leftNode = node.left;
            if (leftNode != null) {
                leftNode.parent = node;
            }
        } else {
            node.right = this._insert(node.right, newNode);
            let rightNode = node.right;
            if (rightNode != null) {
                rightNode.parent = node;
            }
        }
        return node;
    }

    private _rebalance(node: AVLNode | null): void {
        let current = node;
        while (current != null) {
            this._updateBalance(current);

            if (current.balance == 2) {
                if (current.right!.balance == 1) {
                    this._rotateLeft(current);
                } else {
                    this._rotateRightLeft(current);
                }
            } else if (current.balance == -2) {
                if (current.left!.balance == -1) {
                    this._rotateRight(current);
                } else {
                    this._rotateLeftRight(current);
                }
            }
            current = current.parent;
        }
    }

    private _updateBalance(node: AVLNode): void {
        node.balance = this._height(node.right) - this._height(node.left);
    }

    private _rotateLeft(node: AVLNode): void {
        let right = node.right!;
        node.right = right.left;
        right.left = node;
    }

    private _rotateRight(node: AVLNode): void {
        let left = node.left!;
        node.left = left.right;
        left.right = node;
    }

    private _rotateLeftRight(node: AVLNode): void {
        this._rotateLeft(node.left!);
        this._rotateRight(node);
    }
    private _height(node: AVLNode | null): i32 {
        if (node == null) {
            return 0;
        }
        return 1 + max(this._height(node.left), this._height(node.right));
    }
    private _rotateRightLeft(node: AVLNode): void {
        this._rotateRight(node.right!);
        this._rotateLeft(node);
    }
    delete(value: i32): void {
        let node = this._search(this.root, value);
        if (node == null) {
            return;
        }

        this.root = this._delete(this.root, value);

        this._rebalance(node.parent!);
    }
    private _delete(node: AVLNode | null, value: i32): AVLNode | null {
        if (node == null) {
            return null;
        } else if (value < node.value) {
            node.left = this._delete(node.left, value);
        } else if (value > node.value) {
            node.right = this._delete(node.right, value);
        } else {
            if (node.left == null && node.right == null) {
                return null;
            } else if (node.left == null) {
                return node.right;
            } else if (node.right == null) {
                return node.left;
            } else {
                let successor = this._findSuccessor(node);

                node.value = successor.value;

                node.right = this._delete(node.right, successor.value);
            }
        }
        return node;
    }
    private _findSuccessor(node: AVLNode): AVLNode {
        let current = node.right!;
        while (current.left != null) {
            current = current.left;
        }
        return current;
    }
    private _search(node: AVLNode | null, value: i32): AVLNode | null {
        if (node == null) {
            return null;
        } else if (value < node.value) {
            return this._search(node.left, value);
        } else if (value > node.value) {
            return this._search(node.right, value);
        } else {
            return node;
        }
    }

    search(value: i32): AVLNode | null {
        return this._search(this.root, value);
    }
    private _toString(node: AVLNode | null, result: string[]): void {
        if (node == null) {
            return;
        }
        this._toString(node.left, result);
        result.push(node.value.toString() + "(" + this._height(node).toString() + ")");
        this._toString(node.right, result);
    }
    toString(): string {
        let result: string[] = [];
        this._toString(this.root, result);
        return result.join(", ");
    }
}
```