#include <iostream>
#include <cmath>

class Node {
public:
    int key;
    int value;
    Node* next;
    Node* prev;
    
    Node(int k, int v) : key(k), value(v), next(nullptr), prev(nullptr) {}
};

class DoublyLinkedList {
private:
    Node* head;
    Node* tail;
    int size;

public:
    DoublyLinkedList() : head(nullptr), tail(nullptr), size(0) {}
    
    ~DoublyLinkedList() {
        Node* current = head;
        while (current != nullptr) {
            Node* next = current->next;
            delete current;
            current = next;
        }
    }
    
    void insert(int key, int value) {
        Node* newNode = new Node(key, value);
        
        if (!head) {
            head = tail = newNode;
        } else {
            tail->next = newNode;
            newNode->prev = tail;
            tail = newNode;
        }
        size++;
    }
    
    bool remove(int key) {
        Node* current = head;
        
        while (current != nullptr) {
            if (current->key == key) {
                if (current->prev) {
                    current->prev->next = current->next;
                } else {
                    head = current->next;
                }
                
                if (current->next) {
                    current->next->prev = current->prev;
                } else {
                    tail = current->prev;
                }
                
                delete current;
                size--;
                return true;
            }
            current = current->next;
        }
        return false;
    }
    
    Node* find(int key) {
        Node* current = head;
        while (current != nullptr) {
            if (current->key == key) {
                return current;
            }
            current = current->next;
        }
        return nullptr;
    }
    
    int getSize() const {
        return size;
    }
    
    Node* getHead() const {
        return head;
    }
};


class HashTable {
private:
    DoublyLinkedList** table;
    int capacity;
    int size;
    double A;  
    
    
    typedef int (*HashFunction)(int key, int capacity, double A);
    HashFunction hashFunc;

    
    static int multiplicationHash(int key, int capacity, double A) {
        double product = key * A;
        double fractional = product - floor(product);
        return floor(capacity * fractional);
    }
    
    
    static int divisionHash(int key, int capacity, double /* A */) {
        return key % capacity;
    }
    
    void resize(int newCapacity) {
        DoublyLinkedList** newTable = new DoublyLinkedList*[newCapacity];
        for (int i = 0; i < newCapacity; i++) {
            newTable[i] = new DoublyLinkedList();
        }
        
        
        for (int i = 0; i < capacity; i++) {
            Node* current = table[i]->getHead();
            while (current != nullptr) {
                int newIndex = hashFunc(current->key, newCapacity, A);
                newTable[newIndex]->insert(current->key, current->value);
                current = current->next;
            }
            delete table[i];
        }
        
        delete[] table;
        table = newTable;
        capacity = newCapacity;
    }

public:
    HashTable(int initialCapacity = 8, HashFunction func = multiplicationHash) 
        : capacity(initialCapacity), size(0), A((sqrt(5) - 1) / 2), hashFunc(func) {
        table = new DoublyLinkedList*[capacity];
        for (int i = 0; i < capacity; i++) {
            table[i] = new DoublyLinkedList();
        }
    }
    
    ~HashTable() {
        for (int i = 0; i < capacity; i++) {
            delete table[i];
        }
        delete[] table;
    }
    
    void insert(int key, int value) {
        int index = hashFunc(key, capacity, A);
        
        
        Node* existing = table[index]->find(key);
        if (existing) {
            existing->value = value;
            return;
        }
        
        table[index]->insert(key, value);
        size++;
        
        
        if (size > capacity * 0.75) {
            resize(capacity * 2);
        }
    }
    
    bool remove(int key) {
        int index = hashFunc(key, capacity, A);
        bool removed = table[index]->remove(key);
        
        if (removed) {
            size--;
            
            if (size < capacity * 0.25 && capacity > 8) {
                resize(capacity / 2);
            }
        }
        
        return removed;
    }
    
    Node* find(int key) {
        int index = hashFunc(key, capacity, A);
        return table[index]->find(key);
    }
    
    int getSize() const {
        return size;
    }
    
    int getCapacity() const {
        return capacity;
    }
};


int main() {
    HashTable ht;
    
    
    ht.insert(5, 50);
    ht.insert(15, 150);
    ht.insert(25, 250);
    
    
    Node* node = ht.find(15);
    if (node) {
std::cout << "Found key 15, value: " << node->value << std::endl;
    }
    
    
    ht.remove(15);
    
    
    node = ht.find(15);
    if (!node) {
        std::cout << "Key 15 was successfully removed" << std::endl;
    }
    
    
    for (int i = 0; i < 20; i++) {
        ht.insert(i * 10, i * 100);
    }
    
    std::cout << "Final table capacity: " << ht.getCapacity() << std::endl;
    std::cout << "Final table size: " << ht.getSize() << std::endl;
    
    return 0;
}
