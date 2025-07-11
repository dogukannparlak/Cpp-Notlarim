# unique_ptr

`unique_ptr`, C++11 ile gelen modern C++'ın en önemli smart pointer'larından biridir. Tek sahiplik (exclusive ownership) modeli kullanarak otomatik bellek yönetimi sağlar ve raw pointer'ların çoğu kullanım durumunu güvenli hale getirir.

## Temel unique_ptr Kullanımı

### Oluşturma ve Temel İşlemler

```cpp
#include <iostream>
#include <memory>
#include <string>
using namespace std;

class Resource {
private:
    string name;
    int id;
    
public:
    Resource(const string& n, int i) : name(n), id(i) {
        cout << "Resource '" << name << "' oluşturuldu (ID: " << id << ")" << endl;
    }
    
    ~Resource() {
        cout << "Resource '" << name << "' yok edildi (ID: " << id << ")" << endl;
    }
    
    void doWork() const {
        cout << "Resource '" << name << "' çalışıyor..." << endl;
    }
    
    const string& getName() const { return name; }
    int getId() const { return id; }
};

void basicUniquePtrDemo() {
    cout << "=== unique_ptr Temel Kullanımı ===" << endl;
    
    // unique_ptr oluşturma yöntemleri
    cout << "\n--- unique_ptr Oluşturma ---" << endl;
    
    // 1. make_unique kullanımı (C++14, önerilen)
    auto ptr1 = make_unique<Resource>("Resource_1", 1);
    
    // 2. Constructor ile (C++11)
    unique_ptr<Resource> ptr2(new Resource("Resource_2", 2));
    
    // 3. Primitive türler için
    auto intPtr = make_unique<int>(42);
    auto doublePtr = make_unique<double>(3.14159);
    
    cout << "\n--- Temel İşlemler ---" << endl;
    
    // Dereference operations
    ptr1->doWork();
    (*ptr1).doWork();
    
    cout << "intPtr değeri: " << *intPtr << endl;
    cout << "doublePtr değeri: " << *doublePtr << endl;
    
    // get() ile raw pointer erişimi
    Resource* rawPtr = ptr1.get();
    cout << "Raw pointer üzerinden: " << rawPtr->getName() << endl;
    
    // nullptr kontrolü
    if (ptr1) {
        cout << "ptr1 geçerli" << endl;
    }
    
    if (ptr1 != nullptr) {
        cout << "ptr1 null değil" << endl;
    }
    
    // reset() operasyonu
    cout << "\n--- reset() İşlemi ---" << endl;
    ptr2.reset();  // Resource'u yok et, nullptr yap
    
    if (!ptr2) {
        cout << "ptr2 artık nullptr" << endl;
    }
    
    // Yeni resource ile reset
    ptr2.reset(new Resource("Resource_2_New", 22));
    ptr2->doWork();
    
    cout << "\nScope sonu - otomatik cleanup" << endl;
}

int main() {
    basicUniquePtrDemo();
    return 0;
}
```

### Move Semantics ve Ownership Transfer

```cpp
#include <iostream>
#include <memory>
#include <vector>
using namespace std;

class MoveableResource {
private:
    string name;
    vector<int> data;
    
public:
    MoveableResource(const string& n, size_t size) : name(n), data(size) {
        for (size_t i = 0; i < size; i++) {
            data[i] = static_cast<int>(i * i);
        }
        cout << "MoveableResource '" << name << "' oluşturuldu (" 
             << size << " elements)" << endl;
    }
    
    ~MoveableResource() {
        cout << "MoveableResource '" << name << "' yok edildi" << endl;
    }
    
    // Copy constructor/assignment deleted (unique ownership)
    MoveableResource(const MoveableResource&) = delete;
    MoveableResource& operator=(const MoveableResource&) = delete;
    
    // Move constructor
    MoveableResource(MoveableResource&& other) noexcept 
        : name(move(other.name)), data(move(other.data)) {
        cout << "MoveableResource '" << name << "' moved" << endl;
    }
    
    // Move assignment
    MoveableResource& operator=(MoveableResource&& other) noexcept {
        if (this != &other) {
            name = move(other.name);
            data = move(other.data);
            cout << "MoveableResource '" << name << "' move assigned" << endl;
        }
        return *this;
    }
    
    void printInfo() const {
        cout << "MoveableResource '" << name << "' - " 
             << data.size() << " elements" << endl;
    }
    
    const string& getName() const { return name; }
    size_t getDataSize() const { return data.size(); }
};

// unique_ptr döndüren factory function
unique_ptr<MoveableResource> createResource(const string& name, size_t size) {
    cout << "Factory: Creating " << name << endl;
    return make_unique<MoveableResource>(name, size);
}

// unique_ptr alan function
void processResource(unique_ptr<MoveableResource> resource) {
    if (resource) {
        cout << "Processing: ";
        resource->printInfo();
        // resource burada otomatik yok edilir
    }
}

// unique_ptr referansı alan function (ownership transfer olmaz)
void inspectResource(const unique_ptr<MoveableResource>& resource) {
    if (resource) {
        cout << "Inspecting: ";
        resource->printInfo();
        // resource burada yok edilmez
    }
}

void moveSemanticsDemo() {
    cout << "\n=== Move Semantics ve Ownership Transfer ===" << endl;
    
    // Factory function'dan unique_ptr alma
    cout << "--- Factory Function ---" << endl;
    auto resource1 = createResource("Factory_Resource", 100);
    resource1->printInfo();
    
    // Move assignment
    cout << "\n--- Move Assignment ---" << endl;
    unique_ptr<MoveableResource> resource2;
    resource2 = move(resource1);  // Ownership transfer
    
    if (!resource1) {
        cout << "resource1 artık nullptr (moved from)" << endl;
    }
    
    if (resource2) {
        cout << "resource2 ownership aldı" << endl;
        resource2->printInfo();
    }
    
    // Function'a ownership transfer
    cout << "\n--- Function Ownership Transfer ---" << endl;
    auto resource3 = createResource("Transfer_Resource", 50);
    cout << "Ownership transfer öncesi..." << endl;
    
    processResource(move(resource3));  // Ownership transfer
    
    if (!resource3) {
        cout << "resource3 artık nullptr (transferred)" << endl;
    }
    
    // Reference ile inspection (no transfer)
    cout << "\n--- Reference Inspection ---" << endl;
    auto resource4 = createResource("Inspect_Resource", 75);
    
    inspectResource(resource4);  // Ownership transfer olmaz
    
    if (resource4) {
        cout << "resource4 hala sahiplikte" << endl;
        resource4->printInfo();
    }
    
    // Vector of unique_ptr
    cout << "\n--- Vector of unique_ptr ---" << endl;
    vector<unique_ptr<MoveableResource>> resources;
    
    resources.push_back(createResource("Vector_1", 10));
    resources.push_back(createResource("Vector_2", 20));
    resources.push_back(createResource("Vector_3", 30));
    
    cout << "Vector boyutu: " << resources.size() << endl;
    
    for (const auto& res : resources) {
        res->printInfo();
    }
    
    // Move from vector
    cout << "\nVector'den move:" << endl;
    auto movedResource = move(resources[1]);
    movedResource->printInfo();
    
    cout << "Vector element durumu:" << endl;
    for (size_t i = 0; i < resources.size(); i++) {
        if (resources[i]) {
            cout << "  Index " << i << ": " << resources[i]->getName() << endl;
        } else {
            cout << "  Index " << i << ": nullptr (moved)" << endl;
        }
    }
    
    cout << "Scope sonu - otomatik cleanup" << endl;
}

int main() {
    basicUniquePtrDemo();
    moveSemanticsDemo();
    return 0;
}
```

## unique_ptr ile Arrays

### Array Management

```cpp
#include <iostream>
#include <memory>
#include <algorithm>
using namespace std;

void arrayUniquePtrDemo() {
    cout << "\n=== unique_ptr ile Array Yönetimi ===" << endl;
    
    // Array unique_ptr oluşturma
    cout << "--- Array unique_ptr ---" << endl;
    
    // C++14 ve sonrası için make_unique
    auto intArray = make_unique<int[]>(10);
    auto doubleArray = make_unique<double[]>(5);
    
    // C++11 için constructor
    unique_ptr<char[]> charArray(new char[20]);
    
    // Array initialization
    cout << "Array initialization:" << endl;
    for (int i = 0; i < 10; i++) {
        intArray[i] = i * i;
    }
    
    for (int i = 0; i < 5; i++) {
        doubleArray[i] = i * 3.14;
    }
    
    for (int i = 0; i < 19; i++) {
        charArray[i] = 'A' + (i % 26);
    }
    charArray[19] = '\0';
    
    // Array kullanımı
    cout << "intArray: ";
    for (int i = 0; i < 10; i++) {
        cout << intArray[i] << " ";
    }
    cout << endl;
    
    cout << "doubleArray: ";
    for (int i = 0; i < 5; i++) {
        cout << doubleArray[i] << " ";
    }
    cout << endl;
    
    cout << "charArray: " << charArray.get() << endl;
    
    // get() ile raw pointer erişimi
    int* rawIntArray = intArray.get();
    cout << "Raw pointer üzerinden ilk eleman: " << rawIntArray[0] << endl;
    
    // Array reset
    cout << "\n--- Array Reset ---" << endl;
    intArray.reset(new int[5]);  // Eski array yok edilir, yeni oluşturulur
    
    for (int i = 0; i < 5; i++) {
        intArray[i] = (i + 1) * 10;
    }
    
    cout << "Yeni intArray: ";
    for (int i = 0; i < 5; i++) {
        cout << intArray[i] << " ";
    }
    cout << endl;
    
    cout << "Array scope sonu - otomatik cleanup" << endl;
}

// Custom deleter ile unique_ptr
class CustomResource {
private:
    string name;
    
public:
    CustomResource(const string& n) : name(n) {
        cout << "CustomResource '" << name << "' oluşturuldu" << endl;
    }
    
    ~CustomResource() {
        cout << "CustomResource '" << name << "' yok edildi" << endl;
    }
    
    const string& getName() const { return name; }
};

// Custom deleter function
void customDeleter(CustomResource* ptr) {
    cout << "Custom deleter çağrıldı: " << ptr->getName() << endl;
    delete ptr;
}

// Custom deleter class
class ResourceDeleter {
public:
    void operator()(CustomResource* ptr) const {
        cout << "ResourceDeleter operator() çağrıldı: " << ptr->getName() << endl;
        delete ptr;
    }
};

void customDeleterDemo() {
    cout << "\n=== Custom Deleter ile unique_ptr ===" << endl;
    
    // Function deleter
    cout << "--- Function Deleter ---" << endl;
    {
        unique_ptr<CustomResource, decltype(&customDeleter)> 
            ptr1(new CustomResource("Function_Deleter"), customDeleter);
        
        cout << "Resource: " << ptr1->getName() << endl;
        cout << "Scope sonu - custom deleter çağrılacak" << endl;
    }
    
    // Class deleter
    cout << "\n--- Class Deleter ---" << endl;
    {
        unique_ptr<CustomResource, ResourceDeleter> 
            ptr2(new CustomResource("Class_Deleter"));
        
        cout << "Resource: " << ptr2->getName() << endl;
        cout << "Scope sonu - class deleter çağrılacak" << endl;
    }
    
    // Lambda deleter
    cout << "\n--- Lambda Deleter ---" << endl;
    {
        auto lambdaDeleter = [](CustomResource* ptr) {
            cout << "Lambda deleter çağrıldı: " << ptr->getName() << endl;
            delete ptr;
        };
        
        unique_ptr<CustomResource, decltype(lambdaDeleter)> 
            ptr3(new CustomResource("Lambda_Deleter"), lambdaDeleter);
        
        cout << "Resource: " << ptr3->getName() << endl;
        cout << "Scope sonu - lambda deleter çağrılacak" << endl;
    }
}

int main() {
    basicUniquePtrDemo();
    moveSemanticsDemo();
    arrayUniquePtrDemo();
    customDeleterDemo();
    return 0;
}
```

## Performance ve Best Practices

### unique_ptr Performance Analysis

```cpp
#include <iostream>
#include <memory>
#include <chrono>
#include <vector>
using namespace std;
using namespace chrono;

class PerformanceTestObject {
private:
    int data[100];  // Büyüklük simülasyonu
    
public:
    PerformanceTestObject(int value = 0) {
        for (int i = 0; i < 100; i++) {
            data[i] = value + i;
        }
    }
    
    int getValue(int index) const {
        return (index < 100) ? data[index] : 0;
    }
};

void performanceComparison() {
    cout << "\n=== Performance Karşılaştırması ===" << endl;
    
    const int iterations = 1000000;
    const int arraySize = 100;
    
    // Raw pointer performance
    cout << "--- Raw Pointer Performance ---" << endl;
    auto start = high_resolution_clock::now();
    
    for (int i = 0; i < iterations; i++) {
        PerformanceTestObject* obj = new PerformanceTestObject(i);
        volatile int value = obj->getValue(50);  // Optimization'ı önle
        delete obj;
    }
    
    auto end = high_resolution_clock::now();
    auto rawPtrTime = duration_cast<microseconds>(end - start);
    
    // unique_ptr performance
    cout << "--- unique_ptr Performance ---" << endl;
    start = high_resolution_clock::now();
    
    for (int i = 0; i < iterations; i++) {
        auto obj = make_unique<PerformanceTestObject>(i);
        volatile int value = obj->getValue(50);  // Optimization'ı önle
        // Otomatik cleanup
    }
    
    end = high_resolution_clock::now();
    auto uniquePtrTime = duration_cast<microseconds>(end - start);
    
    // Stack allocation performance
    cout << "--- Stack Allocation Performance ---" << endl;
    start = high_resolution_clock::now();
    
    for (int i = 0; i < iterations; i++) {
        PerformanceTestObject obj(i);
        volatile int value = obj.getValue(50);  // Optimization'ı önle
    }
    
    end = high_resolution_clock::now();
    auto stackTime = duration_cast<microseconds>(end - start);
    
    // Results
    cout << "\nPerformance Results (" << iterations << " iterations):" << endl;
    cout << "Raw Pointer:  " << rawPtrTime.count() << " μs" << endl;
    cout << "unique_ptr:   " << uniquePtrTime.count() << " μs" << endl;
    cout << "Stack Alloc:  " << stackTime.count() << " μs" << endl;
    
    cout << "\nOverhead:" << endl;
    cout << "unique_ptr vs Raw: " << 
        (uniquePtrTime.count() / (double)rawPtrTime.count()) << "x" << endl;
    cout << "Heap vs Stack: " << 
        (uniquePtrTime.count() / (double)stackTime.count()) << "x" << endl;
}

// Best practices demonstration
class BestPracticesDemo {
public:
    // Factory function döndürme
    static unique_ptr<PerformanceTestObject> createObject(int value) {
        return make_unique<PerformanceTestObject>(value);
    }
    
    // Const reference ile geçirme (ownership transfer yok)
    static void processObject(const unique_ptr<PerformanceTestObject>& obj) {
        if (obj) {
            cout << "Processing object, value[0]: " << obj->getValue(0) << endl;
        }
    }
    
    // Ownership transfer ile alma
    static void takeOwnership(unique_ptr<PerformanceTestObject> obj) {
        if (obj) {
            cout << "Took ownership, value[1]: " << obj->getValue(1) << endl;
            // obj burada otomatik yok edilir
        }
    }
    
    // Reference döndürme (dikkatli kullanım)
    static const PerformanceTestObject& getReference(
        const unique_ptr<PerformanceTestObject>& obj) {
        
        if (!obj) {
            throw invalid_argument("Null unique_ptr reference");
        }
        return *obj;
    }
};

void bestPracticesDemo() {
    cout << "\n=== unique_ptr Best Practices ===" << endl;
    
    // 1. make_unique kullanın
    cout << "--- make_unique Kullanımı ---" << endl;
    auto obj1 = make_unique<PerformanceTestObject>(100);  // Güvenli
    // unique_ptr<PerformanceTestObject> obj2(new PerformanceTestObject(100));  // Daha az güvenli
    
    // 2. Factory functions
    cout << "\n--- Factory Functions ---" << endl;
    auto obj2 = BestPracticesDemo::createObject(200);
    BestPracticesDemo::processObject(obj2);  // Ownership korunur
    
    // 3. const reference ile parametre geçirme
    cout << "\n--- Parameter Passing ---" << endl;
    BestPracticesDemo::processObject(obj1);  // obj1 hala sahiplikte
    
    if (obj1) {
        cout << "obj1 hala geçerli: " << obj1->getValue(0) << endl;
    }
    
    // 4. Ownership transfer
    cout << "\n--- Ownership Transfer ---" << endl;
    auto obj3 = BestPracticesDemo::createObject(300);
    cout << "Transfer öncesi obj3 geçerli: " << (obj3 != nullptr) << endl;
    
    BestPracticesDemo::takeOwnership(move(obj3));  // Ownership transfer
    cout << "Transfer sonrası obj3 geçerli: " << (obj3 != nullptr) << endl;
    
    // 5. Reference döndürme
    cout << "\n--- Reference Return ---" << endl;
    try {
        const auto& ref = BestPracticesDemo::getReference(obj1);
        cout << "Reference ile erişim: " << ref.getValue(5) << endl;
    } catch (const exception& e) {
        cout << "Hata: " << e.what() << endl;
    }
    
    // 6. Container kullanımı
    cout << "\n--- Container Usage ---" << endl;
    vector<unique_ptr<PerformanceTestObject>> objects;
    
    for (int i = 0; i < 5; i++) {
        objects.push_back(make_unique<PerformanceTestObject>(i * 10));
    }
    
    cout << "Container boyutu: " << objects.size() << endl;
    
    // Container iteration
    for (const auto& obj : objects) {
        cout << "Object value[0]: " << obj->getValue(0) << endl;
    }
    
    // Container'dan move
    auto movedObj = move(objects[2]);
    cout << "Moved object value[0]: " << movedObj->getValue(0) << endl;
    
    cout << "Container element durumu:" << endl;
    for (size_t i = 0; i < objects.size(); i++) {
        if (objects[i]) {
            cout << "  Index " << i << ": geçerli" << endl;
        } else {
            cout << "  Index " << i << ": nullptr (moved)" << endl;
        }
    }
}

int main() {
    basicUniquePtrDemo();
    moveSemanticsDemo();
    arrayUniquePtrDemo();
    customDeleterDemo();
    performanceComparison();
    bestPracticesDemo();
    
    cout << "\n=== unique_ptr Özeti ===" << endl;
    cout << "✓ Otomatik bellek yönetimi" << endl;
    cout << "✓ Zero overhead (optimized builds)" << endl;
    cout << "✓ Exception safe" << endl;
    cout << "✓ Move semantics desteği" << endl;
    cout << "✓ Custom deleter desteği" << endl;
    cout << "✓ Array desteği" << endl;
    cout << "✓ Raw pointer'dan güvenli" << endl;
    cout << "✗ Kopyalanamaz (sadece move)" << endl;
    cout << "✗ Shared ownership yok" << endl;
    cout << "→ Raw pointer yerine unique_ptr kullanın" << endl;
    
    return 0;
}
```

`unique_ptr`, modern C++'da raw pointer'ların yerini alan en önemli araçlardan biridir. Zero overhead ile otomatik bellek yönetimi sağlar ve bellek sızıntılarını önler.
