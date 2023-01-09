# AVL tree for AssemblyScript
## Code
```TypeScript

import { logi, log } from "./env";
import { chkMemAvai } from "../node_modules/asc-linear-rt/lm"
export function main(): void {
    // const ptr = 21354;
    // const ptr2 = __alloc(offsetof<i32>());

    // // i32.store(ptr, 42);
    // i32.store(ptr2, 42);

    // Create a new AVL tree
    let tree = new AVLTree();

    // Insert some values into the tree
    tree.insert(10);
    tree.insert(5);
    tree.insert(15);
    tree.insert(3);
    tree.insert(7);
    tree.insert(13);
    tree.insert(17);
    log(tree.toString());
}

// Define the data structure for the tree nodes
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

    // Insert a new value into the tree
    insert(value: i32): void {
        // Insert the value as a new leaf node
        let newNode = new AVLNode(value);
        this.root = this._insert(this.root, newNode);

        // Rebalance the tree if necessary
        this._rebalance(newNode);
    }

    // Recursive function for inserting a node into the tree
    private _insert(node: AVLNode | null, newNode: AVLNode): AVLNode {
        if (node == null) {
            return newNode;
        } else if (newNode.value < node.value) {
            node.left = this._insert(node.left, newNode);
            let leftNode = node.left;
            if(leftNode !=null) {
                leftNode = node;
            }
        } else {
            node.right = this._insert(node.right, newNode);
            let rightNode = node.right;
            if(rightNode !=null) {
                rightNode = node;
            }
        }
        return node;
    }

    // Rebalance the tree if necessary after inserting a new node
    private _rebalance(node: AVLNode | null): void {
        // Update the balance factor for each node on the path from the new node
        // to the root
        let current = node;
        while (current != null) {
            this._updateBalance(current);

            // If the balance factor becomes 2 or -2, perform a rotation
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

    // Update the balance factor for a given node
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
    // Perform a right-left rotation on a node
    private _rotateRightLeft(node: AVLNode): void {
        this._rotateRight(node.right!);
        this._rotateLeft(node);
    }
    // Delete a value from the tree
    delete(value: i32): void {
        // Find the node containing the value to be deleted
        let node = this._search(this.root, value);
        if (node == null) {
            return;
        }

        // Delete the node
        this.root = this._delete(this.root, value);

        // Rebalance the tree if necessary
        this._rebalance(node.parent!);
    }
    // Recursive function for deleting a node from the tree
    private _delete(node: AVLNode | null, value: i32): AVLNode | null {
        if (node == null) {
            return null;
        } else if (value < node.value) {
            node.left = this._delete(node.left, value);
        } else if (value > node.value) {
            node.right = this._delete(node.right, value);
        } else {
            // Node to be deleted has been found
            if (node.left == null && node.right == null) {
                // Node is a leaf, simply remove it
                return null;
            } else if (node.left == null) {
                // Node has only a right child, replace it with its right child
                return node.right;
            } else if (node.right == null) {
                // Node has only a left child, replace it with its left child
                return node.left;
            } else {
                // Node has two children, find the next in-order successor
                let successor = this._findSuccessor(node);

                // Copy the value of the successor to the current node
                node.value = successor.value;

                // Delete the successor
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
    // Recursive function for searching the tree for a specific value
    private _search(node: AVLNode | null, value: i32): AVLNode | null {
        if (node == null) {
            return null;
        } else if (value < node.value) {
            return this._search(node.left, value);
        } else if (value > node.value) {
            return this._search(node.right, value);
        } else {
            // Value has been found
            return node;
        }
    }

    // Search the tree for a specific value
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