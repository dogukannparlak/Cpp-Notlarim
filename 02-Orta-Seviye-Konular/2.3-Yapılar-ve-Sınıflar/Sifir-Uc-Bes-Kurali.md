# Sıfır, Beş, Üç Kuralı (Rule of Zero/Three/Five)

C++'da resource management ve object lifetime yönetimi için kritik öneme sahip kurallar. Bu kurallar, sınıfların nasıl copy/move edilebileceğini ve resource'ların nasıl yönetileceğini belirler.

## Rule of Zero (Sıfır Kuralı)

### Modern C++ Yaklaşımı

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <fstream>

// Rule of Zero - En iyi yaklaşım
// Özel destructor, copy constructor, copy assignment operator
// move constructor, move assignment operator yazmayın
// STL container'ları ve smart pointer'ları kullanın

class ModernStudent {
private:
    std::string name;
    std::vector<int> grades;
    std::unique_ptr<std::string> notes;
    std::shared_ptr<std::string> sharedData;
    
public:
    // Tek constructor yeterli
    ModernStudent(const std::string& studentName) 
        : name(studentName), 
          notes(std::make_unique<std::string>("Initial notes")),
          sharedData(std::make_shared<std::string>("Shared data")) {
        std::cout << "ModernStudent constructor: " << name << std::endl;
    }
    
    // Destructor, copy/move operations otomatik olarak compiler tarafından generate edilir
    // Çünkü member'lar RAII-compliant
    
    // Public interface methods
    void addGrade(int grade) {
        grades.push_back(grade);
    }
    
    void addNote(const std::string& note) {
        *notes += "\n" + note;
    }
    
    double getAverage() const {
        if (grades.empty()) return 0.0;
        double sum = 0.0;
        for (int grade : grades) {
            sum += grade;
        }
        return sum / grades.size();
    }
    
    void printInfo() const {
        std::cout << "Student: " << name << std::endl;
        std::cout << "Grades: ";
        for (int grade : grades) {
            std::cout << grade << " ";
        }
        std::cout << "\nAverage: " << getAverage() << std::endl;
        std::cout << "Notes: " << *notes << std::endl;
        std::cout << "Shared data: " << *sharedData << std::endl;
        std::cout << "Shared data use count: " << sharedData.use_count() << std::endl;
    }
    
    // Getter methods
    const std::string& getName() const { return name; }
    const std::vector<int>& getGrades() const { return grades; }
    const std::string& getNotes() const { return *notes; }
    std::shared_ptr<std::string> getSharedData() const { return sharedData; }
};

// RAII Resource Manager
class FileManager {
private:
    std::unique_ptr<std::ofstream> file;
    std::string filename;
    
public:
    FileManager(const std::string& fname) : filename(fname) {
        file = std::make_unique<std::ofstream>(filename);
        if (!file->is_open()) {
            throw std::runtime_error("Cannot open file: " + filename);
        }
        std::cout << "FileManager opened: " << filename << std::endl;
    }
    
    // Rule of Zero - destructor otomatik, unique_ptr file'ı otomatik kapatır
    
    void write(const std::string& content) {
        if (file && file->is_open()) {
            *file << content << std::endl;
            file->flush();
        }
    }
    
    void writeLine(const std::string& line) {
        write(line);
    }
    
    bool isOpen() const {
        return file && file->is_open();
    }
    
    const std::string& getFilename() const {
        return filename;
    }
};

// Container wrapper with Rule of Zero
template<typename T>
class SafeContainer {
private:
    std::vector<T> data;
    std::string containerName;
    
public:
    SafeContainer(const std::string& name) : containerName(name) {
        std::cout << "SafeContainer created: " << containerName << std::endl;
    }
    
    // Rule of Zero - vector handles everything
    
    void add(const T& item) {
        data.push_back(item);
    }
    
    void add(T&& item) {
        data.push_back(std::move(item));
    }
    
    const T& get(size_t index) const {
        if (index >= data.size()) {
            throw std::out_of_range("Index out of range");
        }
        return data[index];
    }
    
    T& get(size_t index) {
        if (index >= data.size()) {
            throw std::out_of_range("Index out of range");
        }
        return data[index];
    }
    
    size_t size() const { return data.size(); }
    bool empty() const { return data.empty(); }
    
    void clear() { data.clear(); }
    
    // Iterator support
    auto begin() { return data.begin(); }
    auto end() { return data.end(); }
    auto begin() const { return data.begin(); }
    auto end() const { return data.end(); }
    
    void printInfo() const {
        std::cout << "Container '" << containerName << "' has " << data.size() << " items:" << std::endl;
        for (const auto& item : data) {
            std::cout << "  " << item << std::endl;
        }
    }
};

void ruleOfZeroDemo() {
    std::cout << "=== Rule of Zero Demo ===" << std::endl;
    
    // Modern approach - Rule of Zero
    std::cout << "\n--- Modern Student Example ---" << std::endl;
    
    ModernStudent student1("Alice");
    student1.addGrade(85);
    student1.addGrade(92);
    student1.addGrade(78);
    student1.addNote("Good in mathematics");
    student1.printInfo();
    
    std::cout << "\n--- Copy/Move Operations (Automatic) ---" << std::endl;
    
    // Copy constructor - otomatik olarak deep copy yapar
    ModernStudent student2 = student1;
    student2.addGrade(95);
    student2.addNote("Excellent improvement");
    
    std::cout << "Original student:" << std::endl;
    student1.printInfo();
    std::cout << "\nCopied student:" << std::endl;
    student2.printInfo();
    
    // Move constructor - otomatik olarak efficient move yapar
    ModernStudent student3 = std::move(ModernStudent("Bob"));
    student3.addGrade(88);
    student3.printInfo();
    
    // FileManager example
    std::cout << "\n--- RAII FileManager Example ---" << std::endl;
    {
        FileManager fm("test_output.txt");
        fm.write("Rule of Zero example");
        fm.write("Automatic resource management");
        std::cout << "File is open: " << fm.isOpen() << std::endl;
    } // Destructor otomatik olarak file'ı kapatır
    
    // Container example
    std::cout << "\n--- Safe Container Example ---" << std::endl;
    SafeContainer<std::string> container("StringContainer");
    container.add("Hello");
    container.add("World");
    container.add(std::string("Move semantics"));
    container.printInfo();
    
    // Copy container
    SafeContainer<std::string> container2 = container;
    container2.add("Copied container");
    container2.printInfo();
    
    std::cout << "\nRule of Zero benefits:" << std::endl;
    std::cout << "✓ Otomatik resource management" << std::endl;
    std::cout << "✓ Exception safety" << std::endl;
    std::cout << "✓ Efficient copy/move operations" << std::endl;
    std::cout << "✓ Minimum code, maximum safety" << std::endl;
}

int main() {
    ruleOfZeroDemo();
    return 0;
}
```

## Rule of Three (Üç Kuralı)

### Legacy C++ ve Raw Resource Management

```cpp
#include <iostream>
#include <string>
#include <cstring>

// Rule of Three - Eğer aşağıdakilerden birini tanımlıyorsanız, 
// muhtemelen üçünü de tanımlamanız gerekir:
// 1. Destructor
// 2. Copy constructor  
// 3. Copy assignment operator

class LegacyString {
private:
    char* data;
    size_t length;
    
    void cleanup() {
        if (data) {
            std::cout << "Deleting data: " << data << std::endl;
            delete[] data;
            data = nullptr;
        }
    }
    
public:
    // Constructor
    LegacyString(const char* str = "") {
        length = str ? strlen(str) : 0;
        data = new char[length + 1];
        if (str) {
            strcpy(data, str);
        } else {
            data[0] = '\0';
        }
        std::cout << "LegacyString constructor: '" << data << "'" << std::endl;
    }
    
    // 1. Destructor (Rule of Three - 1/3)
    ~LegacyString() {
        std::cout << "LegacyString destructor called for: '" << (data ? data : "null") << "'" << std::endl;
        cleanup();
    }
    
    // 2. Copy Constructor (Rule of Three - 2/3)
    LegacyString(const LegacyString& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        std::cout << "LegacyString copy constructor: '" << data << "'" << std::endl;
    }
    
    // 3. Copy Assignment Operator (Rule of Three - 3/3)
    LegacyString& operator=(const LegacyString& other) {
        std::cout << "LegacyString copy assignment called" << std::endl;
        
        // Self-assignment check
        if (this == &other) {
            std::cout << "Self-assignment detected, returning *this" << std::endl;
            return *this;
        }
        
        // Clean up existing resource
        cleanup();
        
        // Copy from other
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        
        std::cout << "Assigned: '" << data << "'" << std::endl;
        return *this;
    }
    
    // Utility methods
    const char* c_str() const { return data ? data : ""; }
    size_t size() const { return length; }
    bool empty() const { return length == 0; }
    
    void append(const char* str) {
        if (!str) return;
        
        size_t newLength = length + strlen(str);
        char* newData = new char[newLength + 1];
        
        strcpy(newData, data);
        strcat(newData, str);
        
        cleanup();
        data = newData;
        length = newLength;
        
        std::cout << "Appended, new string: '" << data << "'" << std::endl;
    }
    
    void print() const {
        std::cout << "LegacyString: '" << c_str() << "' (length: " << length << ")" << std::endl;
    }
};

// Daha karmaşık bir örnek - Dynamic Array
template<typename T>
class DynamicArray {
private:
    T* elements;
    size_t capacity;
    size_t count;
    
    void cleanup() {
        if (elements) {
            std::cout << "Cleaning up DynamicArray with " << count << " elements" << std::endl;
            delete[] elements;
            elements = nullptr;
        }
    }
    
public:
    // Constructor
    explicit DynamicArray(size_t initialCapacity = 10) 
        : capacity(initialCapacity), count(0) {
        elements = new T[capacity];
        std::cout << "DynamicArray constructor, capacity: " << capacity << std::endl;
    }
    
    // 1. Destructor (Rule of Three - 1/3)
    ~DynamicArray() {
        std::cout << "DynamicArray destructor called" << std::endl;
        cleanup();
    }
    
    // 2. Copy Constructor (Rule of Three - 2/3)
    DynamicArray(const DynamicArray& other) 
        : capacity(other.capacity), count(other.count) {
        elements = new T[capacity];
        
        // Copy elements
        for (size_t i = 0; i < count; ++i) {
            elements[i] = other.elements[i];
        }
        
        std::cout << "DynamicArray copy constructor, copied " << count << " elements" << std::endl;
    }
    
    // 3. Copy Assignment Operator (Rule of Three - 3/3)
    DynamicArray& operator=(const DynamicArray& other) {
        std::cout << "DynamicArray copy assignment called" << std::endl;
        
        // Self-assignment check
        if (this == &other) {
            std::cout << "Self-assignment detected" << std::endl;
            return *this;
        }
        
        // Clean up existing resources
        cleanup();
        
        // Copy from other
        capacity = other.capacity;
        count = other.count;
        elements = new T[capacity];
        
        for (size_t i = 0; i < count; ++i) {
            elements[i] = other.elements[i];
        }
        
        std::cout << "Assigned " << count << " elements" << std::endl;
        return *this;
    }
    
    // Public interface
    void push_back(const T& element) {
        if (count >= capacity) {
            resize(capacity * 2);
        }
        elements[count++] = element;
        std::cout << "Added element, count: " << count << std::endl;
    }
    
    T& operator[](size_t index) {
        if (index >= count) {
            throw std::out_of_range("Index out of range");
        }
        return elements[index];
    }
    
    const T& operator[](size_t index) const {
        if (index >= count) {
            throw std::out_of_range("Index out of range");
        }
        return elements[index];
    }
    
    size_t size() const { return count; }
    size_t getCapacity() const { return capacity; }
    bool empty() const { return count == 0; }
    
    void print() const {
        std::cout << "DynamicArray [";
        for (size_t i = 0; i < count; ++i) {
            std::cout << elements[i];
            if (i < count - 1) std::cout << ", ";
        }
        std::cout << "] (size: " << count << ", capacity: " << capacity << ")" << std::endl;
    }
    
private:
    void resize(size_t newCapacity) {
        std::cout << "Resizing from " << capacity << " to " << newCapacity << std::endl;
        
        T* newElements = new T[newCapacity];
        
        // Copy existing elements
        for (size_t i = 0; i < count; ++i) {
            newElements[i] = elements[i];
        }
        
        cleanup();
        elements = newElements;
        capacity = newCapacity;
    }
};

// Resource manager with Rule of Three
class ResourceManager {
private:
    int* resourceArray;
    size_t resourceCount;
    std::string managerName;
    
    void cleanup() {
        if (resourceArray) {
            std::cout << "ResourceManager '" << managerName << "' cleaning up " 
                      << resourceCount << " resources" << std::endl;
            delete[] resourceArray;
            resourceArray = nullptr;
        }
    }
    
public:
    ResourceManager(const std::string& name, size_t count) 
        : managerName(name), resourceCount(count) {
        resourceArray = new int[resourceCount];
        
        // Initialize with sequential values
        for (size_t i = 0; i < resourceCount; ++i) {
            resourceArray[i] = static_cast<int>(i + 1);
        }
        
        std::cout << "ResourceManager '" << managerName << "' created with " 
                  << resourceCount << " resources" << std::endl;
    }
    
    ~ResourceManager() {
        std::cout << "ResourceManager '" << managerName << "' destructor" << std::endl;
        cleanup();
    }
    
    ResourceManager(const ResourceManager& other) 
        : managerName(other.managerName + "_copy"), resourceCount(other.resourceCount) {
        resourceArray = new int[resourceCount];
        
        for (size_t i = 0; i < resourceCount; ++i) {
            resourceArray[i] = other.resourceArray[i];
        }
        
        std::cout << "ResourceManager copy constructor: '" << managerName << "'" << std::endl;
    }
    
    ResourceManager& operator=(const ResourceManager& other) {
        std::cout << "ResourceManager copy assignment" << std::endl;
        
        if (this == &other) {
            return *this;
        }
        
        cleanup();
        
        managerName = other.managerName + "_assigned";
        resourceCount = other.resourceCount;
        resourceArray = new int[resourceCount];
        
        for (size_t i = 0; i < resourceCount; ++i) {
            resourceArray[i] = other.resourceArray[i];
        }
        
        std::cout << "Assigned to: '" << managerName << "'" << std::endl;
        return *this;
    }
    
    void print() const {
        std::cout << "ResourceManager '" << managerName << "': [";
        for (size_t i = 0; i < resourceCount; ++i) {
            std::cout << resourceArray[i];
            if (i < resourceCount - 1) std::cout << ", ";
        }
        std::cout << "]" << std::endl;
    }
    
    int& operator[](size_t index) {
        if (index >= resourceCount) {
            throw std::out_of_range("Resource index out of range");
        }
        return resourceArray[index];
    }
    
    const std::string& getName() const { return managerName; }
    size_t getCount() const { return resourceCount; }
};

void ruleOfThreeDemo() {
    std::cout << "\n=== Rule of Three Demo ===" << std::endl;
    
    // LegacyString example
    std::cout << "\n--- LegacyString Example ---" << std::endl;
    
    LegacyString str1("Hello");
    str1.print();
    
    // Copy constructor
    LegacyString str2 = str1;  // Copy constructor
    str2.append(" World");
    str1.print();
    str2.print();
    
    // Copy assignment
    LegacyString str3("Initial");
    str3.print();
    str3 = str2;  // Copy assignment
    str1.print();
    str2.print();
    str3.print();
    
    // Self-assignment test
    str3 = str3;
    str3.print();
    
    // DynamicArray example
    std::cout << "\n--- DynamicArray Example ---" << std::endl;
    
    DynamicArray<int> arr1(5);
    arr1.push_back(10);
    arr1.push_back(20);
    arr1.push_back(30);
    arr1.print();
    
    // Copy constructor
    DynamicArray<int> arr2 = arr1;
    arr2.push_back(40);
    arr1.print();
    arr2.print();
    
    // Copy assignment
    DynamicArray<int> arr3(3);
    arr3.push_back(100);
    arr3.print();
    arr3 = arr2;
    arr3.print();
    
    // ResourceManager example
    std::cout << "\n--- ResourceManager Example ---" << std::endl;
    
    ResourceManager rm1("Manager1", 5);
    rm1.print();
    rm1[2] = 999;
    rm1.print();
    
    ResourceManager rm2 = rm1;  // Copy constructor
    rm2.print();
    
    ResourceManager rm3("Manager3", 3);
    rm3.print();
    rm3 = rm1;  // Copy assignment
    rm3.print();
    
    std::cout << "\nRule of Three principles:" << std::endl;
    std::cout << "✓ Deep copy semantics" << std::endl;
    std::cout << "✓ Self-assignment safety" << std::endl;
    std::cout << "✓ Resource cleanup in destructor" << std::endl;
    std::cout << "✓ Exception safety considerations" << std::endl;
}

int main() {
    ruleOfZeroDemo();
    ruleOfThreeDemo();
    return 0;
}
```

## Rule of Five (Beş Kuralı)

### Modern C++ ile Move Semantics

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <utility>
#include <algorithm>

// Rule of Five - C++11 ve sonrası
// Eğer aşağıdakilerden birini tanımlıyorsanız, beşini de düşünmelisiniz:
// 1. Destructor
// 2. Copy constructor
// 3. Copy assignment operator
// 4. Move constructor
// 5. Move assignment operator

class AdvancedBuffer {
private:
    char* buffer;
    size_t size;
    size_t capacity;
    std::string bufferName;
    
    void cleanup() {
        if (buffer) {
            std::cout << "Cleaning up buffer '" << bufferName << "' (size: " << size << ")" << std::endl;
            delete[] buffer;
            buffer = nullptr;
        }
    }
    
public:
    // Constructor
    AdvancedBuffer(const std::string& name, size_t cap = 1024) 
        : bufferName(name), capacity(cap), size(0) {
        buffer = new char[capacity];
        std::fill(buffer, buffer + capacity, 0);
        std::cout << "AdvancedBuffer '" << bufferName << "' created (capacity: " << capacity << ")" << std::endl;
    }
    
    // 1. Destructor (Rule of Five - 1/5)
    ~AdvancedBuffer() {
        std::cout << "AdvancedBuffer '" << bufferName << "' destructor" << std::endl;
        cleanup();
    }
    
    // 2. Copy Constructor (Rule of Five - 2/5)
    AdvancedBuffer(const AdvancedBuffer& other) 
        : bufferName(other.bufferName + "_copy"), capacity(other.capacity), size(other.size) {
        
        buffer = new char[capacity];
        std::copy(other.buffer, other.buffer + capacity, buffer);
        
        std::cout << "AdvancedBuffer copy constructor: '" << bufferName 
                  << "' (copied " << size << " bytes)" << std::endl;
    }
    
    // 3. Copy Assignment Operator (Rule of Five - 3/5)
    AdvancedBuffer& operator=(const AdvancedBuffer& other) {
        std::cout << "AdvancedBuffer copy assignment" << std::endl;
        
        if (this == &other) {
            std::cout << "Self-assignment detected" << std::endl;
            return *this;
        }
        
        // Clean up current resources
        cleanup();
        
        // Copy from other
        bufferName = other.bufferName + "_assigned";
        capacity = other.capacity;
        size = other.size;
        buffer = new char[capacity];
        std::copy(other.buffer, other.buffer + capacity, buffer);
        
        std::cout << "Copy assigned to: '" << bufferName << "'" << std::endl;
        return *this;
    }
    
    // 4. Move Constructor (Rule of Five - 4/5)
    AdvancedBuffer(AdvancedBuffer&& other) noexcept
        : buffer(other.buffer), capacity(other.capacity), size(other.size), 
          bufferName(std::move(other.bufferName)) {
        
        // Reset other object to safe state
        other.buffer = nullptr;
        other.capacity = 0;
        other.size = 0;
        other.bufferName = "moved_from";
        
        std::cout << "AdvancedBuffer move constructor: '" << bufferName 
                  << "' (moved " << size << " bytes)" << std::endl;
    }
    
    // 5. Move Assignment Operator (Rule of Five - 5/5)
    AdvancedBuffer& operator=(AdvancedBuffer&& other) noexcept {
        std::cout << "AdvancedBuffer move assignment" << std::endl;
        
        if (this == &other) {
            std::cout << "Self-move-assignment detected" << std::endl;
            return *this;
        }
        
        // Clean up current resources
        cleanup();
        
        // Move from other
        buffer = other.buffer;
        capacity = other.capacity;
        size = other.size;
        bufferName = std::move(other.bufferName);
        
        // Reset other object
        other.buffer = nullptr;
        other.capacity = 0;
        other.size = 0;
        other.bufferName = "moved_from";
        
        std::cout << "Move assigned to: '" << bufferName << "'" << std::endl;
        return *this;
    }
    
    // Public interface
    void write(const std::string& data) {
        size_t dataSize = data.size();
        if (size + dataSize >= capacity) {
            std::cout << "Buffer full, cannot write: " << data << std::endl;
            return;
        }
        
        std::copy(data.begin(), data.end(), buffer + size);
        size += dataSize;
        
        std::cout << "Written to '" << bufferName << "': " << data 
                  << " (total size: " << size << ")" << std::endl;
    }
    
    void clear() {
        std::fill(buffer, buffer + capacity, 0);
        size = 0;
        std::cout << "Buffer '" << bufferName << "' cleared" << std::endl;
    }
    
    std::string read() const {
        if (!buffer || size == 0) return "";
        return std::string(buffer, size);
    }
    
    // Status methods
    const std::string& getName() const { return bufferName; }
    size_t getSize() const { return size; }
    size_t getCapacity() const { return capacity; }
    bool empty() const { return size == 0; }
    bool isValid() const { return buffer != nullptr; }
    
    void printStatus() const {
        std::cout << "Buffer '" << bufferName << "': " 
                  << (isValid() ? "valid" : "invalid") 
                  << ", size: " << size 
                  << ", capacity: " << capacity;
        if (isValid() && size > 0) {
            std::cout << ", content: '" << read() << "'";
        }
        std::cout << std::endl;
    }
};

// Smart wrapper with Rule of Five
template<typename T>
class SmartWrapper {
private:
    T* ptr;
    std::string wrapperName;
    
public:
    // Constructor
    SmartWrapper(const std::string& name, const T& value) 
        : wrapperName(name), ptr(new T(value)) {
        std::cout << "SmartWrapper '" << wrapperName << "' created" << std::endl;
    }
    
    // Destructor
    ~SmartWrapper() {
        std::cout << "SmartWrapper '" << wrapperName << "' destructor" << std::endl;
        delete ptr;
    }
    
    // Copy constructor
    SmartWrapper(const SmartWrapper& other) 
        : wrapperName(other.wrapperName + "_copy"), ptr(new T(*other.ptr)) {
        std::cout << "SmartWrapper copy constructor: '" << wrapperName << "'" << std::endl;
    }
    
    // Copy assignment
    SmartWrapper& operator=(const SmartWrapper& other) {
        std::cout << "SmartWrapper copy assignment" << std::endl;
        
        if (this == &other) return *this;
        
        delete ptr;
        wrapperName = other.wrapperName + "_assigned";
        ptr = new T(*other.ptr);
        
        std::cout << "Copy assigned to: '" << wrapperName << "'" << std::endl;
        return *this;
    }
    
    // Move constructor
    SmartWrapper(SmartWrapper&& other) noexcept 
        : ptr(other.ptr), wrapperName(std::move(other.wrapperName)) {
        other.ptr = nullptr;
        other.wrapperName = "moved_from";
        std::cout << "SmartWrapper move constructor: '" << wrapperName << "'" << std::endl;
    }
    
    // Move assignment
    SmartWrapper& operator=(SmartWrapper&& other) noexcept {
        std::cout << "SmartWrapper move assignment" << std::endl;
        
        if (this == &other) return *this;
        
        delete ptr;
        ptr = other.ptr;
        wrapperName = std::move(other.wrapperName);
        
        other.ptr = nullptr;
        other.wrapperName = "moved_from";
        
        std::cout << "Move assigned to: '" << wrapperName << "'" << std::endl;
        return *this;
    }
    
    // Interface
    T& get() {
        if (!ptr) throw std::runtime_error("Accessing moved-from object");
        return *ptr;
    }
    
    const T& get() const {
        if (!ptr) throw std::runtime_error("Accessing moved-from object");
        return *ptr;
    }
    
    bool isValid() const { return ptr != nullptr; }
    const std::string& getName() const { return wrapperName; }
    
    void printStatus() const {
        std::cout << "SmartWrapper '" << wrapperName << "': " 
                  << (isValid() ? "valid" : "invalid");
        if (isValid()) {
            std::cout << ", value: " << *ptr;
        }
        std::cout << std::endl;
    }
};

// Helper function to create temporary objects
AdvancedBuffer createTempBuffer(const std::string& name, size_t capacity) {
    AdvancedBuffer temp(name, capacity);
    temp.write("Temporary data");
    return temp;  // Move constructor will be called
}

template<typename T>
SmartWrapper<T> createTempWrapper(const std::string& name, const T& value) {
    SmartWrapper<T> temp(name, value);
    return temp;  // Move constructor will be called
}

void ruleOfFiveDemo() {
    std::cout << "\n=== Rule of Five Demo ===" << std::endl;
    
    // AdvancedBuffer examples
    std::cout << "\n--- AdvancedBuffer Example ---" << std::endl;
    
    AdvancedBuffer buf1("Buffer1", 100);
    buf1.write("Hello");
    buf1.write(" World");
    buf1.printStatus();
    
    // Copy operations
    std::cout << "\n--- Copy Operations ---" << std::endl;
    AdvancedBuffer buf2 = buf1;  // Copy constructor
    buf2.write("!");
    buf1.printStatus();
    buf2.printStatus();
    
    AdvancedBuffer buf3("Buffer3", 50);
    buf3.write("Initial");
    buf3.printStatus();
    buf3 = buf1;  // Copy assignment
    buf3.printStatus();
    
    // Move operations
    std::cout << "\n--- Move Operations ---" << std::endl;
    AdvancedBuffer buf4 = std::move(buf2);  // Move constructor
    buf2.printStatus();  // buf2 is now in moved-from state
    buf4.printStatus();
    
    AdvancedBuffer buf5("Buffer5", 80);
    buf5.write("Will be overwritten");
    buf5.printStatus();
    buf5 = std::move(buf4);  // Move assignment
    buf4.printStatus();  // buf4 is now in moved-from state
    buf5.printStatus();
    
    // Function return (move)
    std::cout << "\n--- Function Return (Move) ---" << std::endl;
    AdvancedBuffer buf6 = createTempBuffer("TempBuffer", 200);
    buf6.printStatus();
    
    // SmartWrapper examples
    std::cout << "\n--- SmartWrapper Example ---" << std::endl;
    
    SmartWrapper<std::string> wrap1("Wrapper1", std::string("Hello"));
    wrap1.printStatus();
    
    // Copy operations
    SmartWrapper<std::string> wrap2 = wrap1;  // Copy constructor
    wrap1.printStatus();
    wrap2.printStatus();
    
    SmartWrapper<std::string> wrap3("Wrapper3", std::string("Initial"));
    wrap3.printStatus();
    wrap3 = wrap1;  // Copy assignment
    wrap3.printStatus();
    
    // Move operations
    SmartWrapper<std::string> wrap4 = std::move(wrap2);  // Move constructor
    wrap2.printStatus();
    wrap4.printStatus();
    
    wrap4 = std::move(wrap3);  // Move assignment
    wrap3.printStatus();
    wrap4.printStatus();
    
    // Function return with move
    SmartWrapper<int> wrap5 = createTempWrapper("TempWrapper", 42);
    wrap5.printStatus();
    
    // Vector with move semantics
    std::cout << "\n--- Vector with Move Semantics ---" << std::endl;
    std::vector<AdvancedBuffer> buffers;
    
    // emplace_back uses move/copy efficiently
    buffers.emplace_back("VectorBuffer1", 50);
    buffers.emplace_back("VectorBuffer2", 60);
    
    // push_back with move
    AdvancedBuffer tempBuf("TempForVector", 70);
    tempBuf.write("Vector data");
    buffers.push_back(std::move(tempBuf));
    tempBuf.printStatus();  // moved-from state
    
    std::cout << "Vector contents:" << std::endl;
    for (const auto& buf : buffers) {
        buf.printStatus();
    }
    
    std::cout << "\nRule of Five benefits:" << std::endl;
    std::cout << "✓ Efficient move operations" << std::endl;
    std::cout << "✓ Exception safety with noexcept" << std::endl;
    std::cout << "✓ STL container compatibility" << std::endl;
    std::cout << "✓ Prevents unnecessary copies" << std::endl;
    std::cout << "✓ Clear ownership semantics" << std::endl;
}

int main() {
    ruleOfZeroDemo();
    ruleOfThreeDemo();
    ruleOfFiveDemo();
    
    std::cout << "\n=== Rules Summary ===" << std::endl;
    std::cout << "\nRule of Zero (Önerilen):" << std::endl;
    std::cout << "✓ RAII-compliant members kullanın (std::vector, std::unique_ptr)" << std::endl;
    std::cout << "✓ Special member functions yazmayın" << std::endl;
    std::cout << "✓ Compiler-generated operations yeterli" << std::endl;
    
    std::cout << "\nRule of Three (Legacy C++):" << std::endl;
    std::cout << "✓ Raw pointer/resource yönetimi gerekli" << std::endl;
    std::cout << "✓ Destructor + Copy constructor + Copy assignment" << std::endl;
    std::cout << "✓ Deep copy semantics" << std::endl;
    
    std::cout << "\nRule of Five (Modern C++):" << std::endl;
    std::cout << "✓ Rule of Three + Move constructor + Move assignment" << std::endl;
    std::cout << "✓ Performance optimizasyonu" << std::endl;
    std::cout << "✓ noexcept kullanın" << std::endl;
    std::cout << "✓ Move-only types için essential" << std::endl;
    
    return 0;
}
```

Bu kurallar, C++'da güvenli ve efficient resource management için kritik öneme sahiptir. Modern C++'da Rule of Zero tercih edilmeli, ancak raw resource management gerektiğinde Rule of Three veya Five doğru şekilde uygulanmalıdır.
