# Erişim İhlalleri (Access Violations)

Erişim ihlalleri, C++ programlarında karşılaşılan en kritik runtime hatalarıdır. Bu hatalar genellikle geçersiz bellek erişimleri, pointer kullanım hataları ve buffer overflow'lar sonucu oluşur. Bu konuları anlamak ve önlemek, güvenli ve stabil C++ kodu yazabilmek için kritik öneme sahiptir.

## Memory Access Violations

### Genel Erişim İhlali Türleri

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <cstring>
#include <csignal>

// Signal handler for crash detection (Unix/Linux systems)
void signalHandler(int signal) {
    std::cout << "\n!!! SIGNAL CAUGHT: ";
    switch (signal) {
        case SIGSEGV:
            std::cout << "SIGSEGV (Segmentation Fault) - Invalid memory access" << std::endl;
            break;
        case SIGABRT:
            std::cout << "SIGABRT (Abort) - Program terminated abnormally" << std::endl;
            break;
        case SIGFPE:
            std::cout << "SIGFPE (Floating Point Exception) - Arithmetic error" << std::endl;
            break;
        default:
            std::cout << "Unknown signal (" << signal << ")" << std::endl;
    }
    std::cout << "Program will exit to prevent further damage." << std::endl;
    exit(1);
}

class AccessViolationDemo {
public:
    // 1. Null Pointer Dereference
    static void demonstrateNullPointerDereference() {
        std::cout << "\n=== Null Pointer Dereference Demo ===" << std::endl;
        
        // Safe example with checking
        int* ptr = nullptr;
        std::cout << "Pointer value: " << ptr << std::endl;
        
        if (ptr != nullptr) {
            std::cout << "Safe access: " << *ptr << std::endl;
        } else {
            std::cout << "✓ Null pointer detected - avoiding dereference" << std::endl;
        }
        
        // Unsafe example (commented out to prevent crash)
        /*
        std::cout << "Attempting unsafe null pointer dereference..." << std::endl;
        int* nullPtr = nullptr;
        std::cout << "Value: " << *nullPtr << std::endl;  // CRASH!
        */
        
        // Real-world scenario - function returning null
        auto getData = []() -> int* {
            // Simulate failure condition
            static bool shouldFail = true;
            if (shouldFail) {
                std::cout << "getData() returning null due to error" << std::endl;
                return nullptr;
            }
            static int data = 42;
            return &data;
        };
        
        int* result = getData();
        if (result) {
            std::cout << "Data retrieved: " << *result << std::endl;
        } else {
            std::cout << "✓ getData() returned null - handled gracefully" << std::endl;
        }
    }
    
    // 2. Dangling Pointer Access
    static void demonstrateDanglingPointer() {
        std::cout << "\n=== Dangling Pointer Demo ===" << std::endl;
        
        int* ptr;
        
        // Create a scope where variable exists
        {
            int localVar = 100;
            ptr = &localVar;
            std::cout << "Within scope - ptr points to: " << *ptr << std::endl;
        } // localVar goes out of scope here
        
        // ptr is now dangling - points to invalid memory
        std::cout << "After scope - ptr still points to: " << static_cast<void*>(ptr) << std::endl;
        std::cout << "✓ Avoiding dangling pointer dereference" << std::endl;
        
        // Unsafe access (commented out to prevent crash)
        /*
        std::cout << "Dangling pointer value: " << *ptr << std::endl;  // UNDEFINED BEHAVIOR!
        */
        
        // Better approach - nullify pointers when objects are destroyed
        ptr = nullptr;
        if (ptr) {
            std::cout << "Safe access: " << *ptr << std::endl;
        } else {
            std::cout << "✓ Pointer nullified - safe from dangling access" << std::endl;
        }
    }
    
    // 3. Buffer Overflow
    static void demonstrateBufferOverflow() {
        std::cout << "\n=== Buffer Overflow Demo ===" << std::endl;
        
        // Stack buffer overflow
        char buffer[10];
        const char* safeData = "Hello";
        const char* unsafeData = "This string is definitely too long for the buffer";
        
        // Safe copy
        std::cout << "Safe string length: " << strlen(safeData) << std::endl;
        if (strlen(safeData) < sizeof(buffer)) {
            strcpy(buffer, safeData);
            std::cout << "✓ Safe copy: " << buffer << std::endl;
        }
        
        // Unsafe copy (commented out to prevent crash)
        std::cout << "Unsafe string length: " << strlen(unsafeData) << std::endl;
        /*
        strcpy(buffer, unsafeData);  // BUFFER OVERFLOW!
        std::cout << "Buffer after overflow: " << buffer << std::endl;
        */
        
        // Safe alternatives
        std::cout << "\n--- Safe Alternatives ---" << std::endl;
        
        // 1. Use strncpy
        char safeBuffer1[10];
        strncpy(safeBuffer1, unsafeData, sizeof(safeBuffer1) - 1);
        safeBuffer1[sizeof(safeBuffer1) - 1] = '\0';  // Ensure null termination
        std::cout << "✓ strncpy result: " << safeBuffer1 << std::endl;
        
        // 2. Use std::string (recommended)
        std::string safeString = unsafeData;
        std::cout << "✓ std::string: " << safeString << std::endl;
        
        // 3. Use vector for dynamic arrays
        std::vector<char> dynamicBuffer(strlen(unsafeData) + 1);
        strcpy(dynamicBuffer.data(), unsafeData);
        std::cout << "✓ vector buffer: " << dynamicBuffer.data() << std::endl;
    }
    
    // 4. Array Bounds Violation
    static void demonstrateArrayBoundsViolation() {
        std::cout << "\n=== Array Bounds Violation Demo ===" << std::endl;
        
        int array[5] = {1, 2, 3, 4, 5};
        
        // Safe access
        std::cout << "Array contents (safe access):" << std::endl;
        for (int i = 0; i < 5; i++) {
            std::cout << "  array[" << i << "] = " << array[i] << std::endl;
        }
        
        // Bounds checking function
        auto safeArrayAccess = [](int* arr, size_t size, int index) -> int* {
            if (index >= 0 && index < static_cast<int>(size)) {
                return &arr[index];
            }
            std::cout << "✗ Index " << index << " out of bounds [0, " << size - 1 << "]" << std::endl;
            return nullptr;
        };
        
        // Test safe access
        std::vector<int> testIndices = {0, 2, 4, 5, -1, 10};
        for (int idx : testIndices) {
            int* result = safeArrayAccess(array, 5, idx);
            if (result) {
                std::cout << "✓ array[" << idx << "] = " << *result << std::endl;
            }
        }
        
        // Unsafe access (commented out to prevent crash)
        /*
        std::cout << "Accessing array[10]: " << array[10] << std::endl;  // OUT OF BOUNDS!
        std::cout << "Accessing array[-1]: " << array[-1] << std::endl;  // OUT OF BOUNDS!
        */
        
        // STL containers with bounds checking
        std::vector<int> safeVector = {1, 2, 3, 4, 5};
        
        try {
            std::cout << "✓ safeVector.at(2) = " << safeVector.at(2) << std::endl;
            std::cout << "Attempting safeVector.at(10)..." << std::endl;
            std::cout << safeVector.at(10) << std::endl;  // Will throw exception
        } catch (const std::out_of_range& e) {
            std::cout << "✓ Exception caught: " << e.what() << std::endl;
        }
    }
    
    // 5. Double Free / Use After Free
    static void demonstrateDoubleFreeProblem() {
        std::cout << "\n=== Double Free / Use After Free Demo ===" << std::endl;
        
        // Manual memory management problems
        int* ptr = new int(42);
        std::cout << "Allocated memory, value: " << *ptr << std::endl;
        
        delete ptr;
        std::cout << "✓ Memory freed successfully" << std::endl;
        
        // Problem 1: Use after free (commented out to prevent crash)
        /*
        std::cout << "Use after free: " << *ptr << std::endl;  // UNDEFINED BEHAVIOR!
        */
        
        // Problem 2: Double free (commented out to prevent crash)
        /*
        delete ptr;  // DOUBLE FREE ERROR!
        */
        
        // Solution: Set pointer to null after deletion
        ptr = nullptr;
        
        if (ptr) {
            std::cout << "Safe access: " << *ptr << std::endl;
        } else {
            std::cout << "✓ Pointer is null - avoiding use after free" << std::endl;
        }
        
        // Attempting to delete null pointer is safe
        delete ptr;  // Safe - deleting null pointer is a no-op
        std::cout << "✓ Deleting null pointer is safe" << std::endl;
        
        // Best practice: Use smart pointers
        std::cout << "\n--- Smart Pointer Alternative ---" << std::endl;
        {
            std::unique_ptr<int> smartPtr = std::make_unique<int>(42);
            std::cout << "Smart pointer value: " << *smartPtr << std::endl;
        } // Automatic cleanup - no manual delete needed
        std::cout << "✓ Smart pointer automatically cleaned up" << std::endl;
    }
};

// Memory debugging utilities
class MemoryDebugger {
private:
    static size_t totalAllocations;
    static size_t totalDeallocations;
    static size_t currentAllocations;
    
public:
    static void* debugMalloc(size_t size, const char* file, int line) {
        void* ptr = malloc(size);
        if (ptr) {
            totalAllocations++;
            currentAllocations++;
            std::cout << "[ALLOC] " << ptr << " (" << size << " bytes) at " 
                      << file << ":" << line << std::endl;
        }
        return ptr;
    }
    
    static void debugFree(void* ptr, const char* file, int line) {
        if (ptr) {
            totalDeallocations++;
            currentAllocations--;
            std::cout << "[FREE] " << ptr << " at " << file << ":" << line << std::endl;
        }
        free(ptr);
    }
    
    static void printStats() {
        std::cout << "\n=== Memory Debug Statistics ===" << std::endl;
        std::cout << "Total allocations: " << totalAllocations << std::endl;
        std::cout << "Total deallocations: " << totalDeallocations << std::endl;
        std::cout << "Current allocations: " << currentAllocations << std::endl;
        std::cout << "Memory leaks: " << (currentAllocations > 0 ? "YES" : "NO") << std::endl;
    }
    
    static void reset() {
        totalAllocations = 0;
        totalDeallocations = 0;
        currentAllocations = 0;
    }
};

size_t MemoryDebugger::totalAllocations = 0;
size_t MemoryDebugger::totalDeallocations = 0;
size_t MemoryDebugger::currentAllocations = 0;

// Macros for memory debugging
#define DEBUG_MALLOC(size) MemoryDebugger::debugMalloc(size, __FILE__, __LINE__)
#define DEBUG_FREE(ptr) MemoryDebugger::debugFree(ptr, __FILE__, __LINE__)

void demonstrateMemoryDebugging() {
    std::cout << "\n=== Memory Debugging Demo ===" << std::endl;
    
    MemoryDebugger::reset();
    
    // Simulate various memory operations
    void* ptr1 = DEBUG_MALLOC(100);
    void* ptr2 = DEBUG_MALLOC(200);
    void* ptr3 = DEBUG_MALLOC(50);
    
    MemoryDebugger::printStats();
    
    DEBUG_FREE(ptr1);
    DEBUG_FREE(ptr2);
    // Intentionally not freeing ptr3 to show leak detection
    
    MemoryDebugger::printStats();
}

int main() {
    // Install signal handlers for crash detection
    signal(SIGSEGV, signalHandler);
    signal(SIGABRT, signalHandler);
    signal(SIGFPE, signalHandler);
    
    std::cout << "=== Access Violations and Memory Safety Demo ===" << std::endl;
    std::cout << "Note: Dangerous operations are commented out to prevent crashes" << std::endl;
    
    try {
        AccessViolationDemo::demonstrateNullPointerDereference();
        AccessViolationDemo::demonstrateDanglingPointer();
        AccessViolationDemo::demonstrateBufferOverflow();
        AccessViolationDemo::demonstrateArrayBoundsViolation();
        AccessViolationDemo::demonstrateDoubleFreeProblem();
        
        demonstrateMemoryDebugging();
        
    } catch (const std::exception& e) {
        std::cout << "Exception caught: " << e.what() << std::endl;
    }
    
    return 0;
}
```

## Debugging Tools ve Techniques

### Advanced Memory Error Detection

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <map>
#include <chrono>
#include <iomanip>
#include <sstream>

// Custom memory allocator with debugging capabilities
class DebuggingAllocator {
private:
    struct AllocationInfo {
        size_t size;
        std::chrono::system_clock::time_point timestamp;
        std::string location;
        bool isActive;
        
        AllocationInfo(size_t s, const std::string& loc) 
            : size(s), timestamp(std::chrono::system_clock::now()), 
              location(loc), isActive(true) {}
    };
    
    static std::map<void*, AllocationInfo> allocations;
    static size_t totalBytesAllocated;
    static size_t totalBytesFreed;
    static size_t peakMemoryUsage;
    static size_t currentMemoryUsage;
    static bool trackingEnabled;
    
public:
    static void enableTracking() { trackingEnabled = true; }
    static void disableTracking() { trackingEnabled = false; }
    
    static void* allocate(size_t size, const std::string& location = "Unknown") {
        void* ptr = std::malloc(size);
        
        if (ptr && trackingEnabled) {
            allocations[ptr] = AllocationInfo(size, location);
            totalBytesAllocated += size;
            currentMemoryUsage += size;
            
            if (currentMemoryUsage > peakMemoryUsage) {
                peakMemoryUsage = currentMemoryUsage;
            }
            
            std::cout << "[ALLOC] " << ptr << " (" << size << " bytes) at " << location << std::endl;
        }
        
        return ptr;
    }
    
    static void deallocate(void* ptr, const std::string& location = "Unknown") {
        if (!ptr) {
            std::cout << "[FREE] Attempt to free null pointer at " << location << std::endl;
            return;
        }
        
        if (trackingEnabled) {
            auto it = allocations.find(ptr);
            if (it != allocations.end()) {
                if (it->second.isActive) {
                    it->second.isActive = false;
                    totalBytesFreed += it->second.size;
                    currentMemoryUsage -= it->second.size;
                    std::cout << "[FREE] " << ptr << " (" << it->second.size << " bytes) at " << location << std::endl;
                } else {
                    std::cout << "[ERROR] Double free detected! " << ptr << " at " << location << std::endl;
                    std::cout << "  Originally allocated at: " << it->second.location << std::endl;
                    return;  // Don't actually free to prevent crash
                }
            } else {
                std::cout << "[ERROR] Attempt to free untracked pointer " << ptr << " at " << location << std::endl;
                return;  // Don't free untracked memory
            }
        }
        
        std::free(ptr);
    }
    
    static void checkForLeaks() {
        std::cout << "\n=== Memory Leak Detection ===" << std::endl;
        
        bool hasLeaks = false;
        size_t leakedBytes = 0;
        
        for (const auto& pair : allocations) {
            if (pair.second.isActive) {
                if (!hasLeaks) {
                    std::cout << "Memory leaks detected:" << std::endl;
                    hasLeaks = true;
                }
                
                auto time_t = std::chrono::system_clock::to_time_t(pair.second.timestamp);
                std::cout << "  LEAK: " << pair.first 
                          << " (" << pair.second.size << " bytes) "
                          << "allocated at " << pair.second.location
                          << " on " << std::ctime(&time_t);
                leakedBytes += pair.second.size;
            }
        }
        
        if (!hasLeaks) {
            std::cout << "✓ No memory leaks detected!" << std::endl;
        } else {
            std::cout << "Total leaked: " << leakedBytes << " bytes" << std::endl;
        }
    }
    
    static void printStatistics() {
        std::cout << "\n=== Memory Statistics ===" << std::endl;
        std::cout << "Total allocations: " << allocations.size() << std::endl;
        std::cout << "Total bytes allocated: " << totalBytesAllocated << std::endl;
        std::cout << "Total bytes freed: " << totalBytesFreed << std::endl;
        std::cout << "Current memory usage: " << currentMemoryUsage << std::endl;
        std::cout << "Peak memory usage: " << peakMemoryUsage << std::endl;
        
        size_t activeAllocations = 0;
        for (const auto& pair : allocations) {
            if (pair.second.isActive) {
                activeAllocations++;
            }
        }
        std::cout << "Active allocations: " << activeAllocations << std::endl;
    }
    
    static void reset() {
        allocations.clear();
        totalBytesAllocated = 0;
        totalBytesFreed = 0;
        peakMemoryUsage = 0;
        currentMemoryUsage = 0;
    }
};

// Static member definitions
std::map<void*, DebuggingAllocator::AllocationInfo> DebuggingAllocator::allocations;
size_t DebuggingAllocator::totalBytesAllocated = 0;
size_t DebuggingAllocator::totalBytesFreed = 0;
size_t DebuggingAllocator::peakMemoryUsage = 0;
size_t DebuggingAllocator::currentMemoryUsage = 0;
bool DebuggingAllocator::trackingEnabled = true;

// Memory debugging macros
#define TRACKED_MALLOC(size) DebuggingAllocator::allocate(size, __FILE__ ":" + std::to_string(__LINE__))
#define TRACKED_FREE(ptr) DebuggingAllocator::deallocate(ptr, __FILE__ ":" + std::to_string(__LINE__))

// Stack overflow detection
class StackAnalyzer {
private:
    static const size_t MAX_RECURSION_DEPTH = 1000;
    static size_t currentDepth;
    
public:
    class StackGuard {
    private:
        bool isValid;
        
    public:
        StackGuard() : isValid(false) {
            if (currentDepth >= MAX_RECURSION_DEPTH) {
                std::cout << "⚠️  Stack overflow risk detected! Depth: " << currentDepth << std::endl;
                return;
            }
            currentDepth++;
            isValid = true;
        }
        
        ~StackGuard() {
            if (isValid) {
                currentDepth--;
            }
        }
        
        bool isSafe() const { return isValid; }
    };
    
    static size_t getCurrentDepth() { return currentDepth; }
    
    // Recursive function to test stack overflow detection
    static void recursiveFunction(int count) {
        StackGuard guard;
        if (!guard.isSafe()) {
            std::cout << "Recursive function stopped due to stack safety" << std::endl;
            return;
        }
        
        std::cout << "Recursion depth: " << getCurrentDepth() << ", count: " << count << std::endl;
        
        if (count > 0) {
            recursiveFunction(count - 1);
        }
    }
};

size_t StackAnalyzer::currentDepth = 0;

// Buffer overflow detection
class BufferOverflowDetector {
private:
    static const uint32_t CANARY_VALUE = 0xDEADBEEF;
    
    struct ProtectedBuffer {
        uint32_t startCanary;
        char* data;
        size_t size;
        uint32_t endCanary;
        
        ProtectedBuffer(size_t bufferSize) : size(bufferSize) {
            startCanary = CANARY_VALUE;
            endCanary = CANARY_VALUE;
            
            // Allocate extra space for canaries
            char* rawMemory = static_cast<char*>(TRACKED_MALLOC(size + 2 * sizeof(uint32_t)));
            
            // Place canaries before and after buffer
            *reinterpret_cast<uint32_t*>(rawMemory) = CANARY_VALUE;
            data = rawMemory + sizeof(uint32_t);
            *reinterpret_cast<uint32_t*>(data + size) = CANARY_VALUE;
        }
        
        ~ProtectedBuffer() {
            checkIntegrity();
            TRACKED_FREE(data - sizeof(uint32_t));
        }
        
        bool checkIntegrity() const {
            uint32_t actualStartCanary = *reinterpret_cast<uint32_t*>(data - sizeof(uint32_t));
            uint32_t actualEndCanary = *reinterpret_cast<uint32_t*>(data + size);
            
            bool isIntact = (startCanary == CANARY_VALUE) && 
                           (endCanary == CANARY_VALUE) &&
                           (actualStartCanary == CANARY_VALUE) && 
                           (actualEndCanary == CANARY_VALUE);
            
            if (!isIntact) {
                std::cout << "🚨 Buffer overflow detected!" << std::endl;
                std::cout << "  Expected start canary: 0x" << std::hex << CANARY_VALUE << std::endl;
                std::cout << "  Actual start canary: 0x" << std::hex << actualStartCanary << std::endl;
                std::cout << "  Expected end canary: 0x" << std::hex << CANARY_VALUE << std::endl;
                std::cout << "  Actual end canary: 0x" << std::hex << actualEndCanary << std::dec << std::endl;
            }
            
            return isIntact;
        }
        
        char* getData() { return data; }
        size_t getSize() const { return size; }
    };
    
public:
    static void demonstrateBufferProtection() {
        std::cout << "\n=== Buffer Overflow Detection Demo ===" << std::endl;
        
        ProtectedBuffer buffer(10);
        
        // Safe operation
        std::cout << "Performing safe buffer operation..." << std::endl;
        strcpy(buffer.getData(), "Hello");
        buffer.checkIntegrity();
        std::cout << "✓ Buffer integrity maintained" << std::endl;
        
        // Simulated overflow (commented out to prevent actual overflow)
        /*
        std::cout << "Simulating buffer overflow..." << std::endl;
        strcpy(buffer.getData(), "This string is way too long for the buffer");
        buffer.checkIntegrity();
        */
        
        std::cout << "Buffer contents: " << buffer.getData() << std::endl;
    }
};

// Use-after-free detection
class UseAfterFreeDetector {
private:
    struct GuardedPointer {
        void* originalPtr;
        size_t size;
        bool isFreed;
        std::string location;
        
        GuardedPointer(void* ptr, size_t s, const std::string& loc) 
            : originalPtr(ptr), size(s), isFreed(false), location(loc) {}
    };
    
    static std::map<void*, GuardedPointer> guardedPointers;
    
public:
    static void* allocateGuarded(size_t size, const std::string& location) {
        void* ptr = TRACKED_MALLOC(size);
        if (ptr) {
            guardedPointers[ptr] = GuardedPointer(ptr, size, location);
        }
        return ptr;
    }
    
    static void freeGuarded(void* ptr, const std::string& location) {
        auto it = guardedPointers.find(ptr);
        if (it != guardedPointers.end()) {
            if (it->second.isFreed) {
                std::cout << "🚨 Use-after-free detected!" << std::endl;
                std::cout << "  Pointer: " << ptr << std::endl;
                std::cout << "  Originally freed at: " << it->second.location << std::endl;
                std::cout << "  Current free attempt at: " << location << std::endl;
                return;
            }
            
            it->second.isFreed = true;
            it->second.location = location;
            
            // Fill memory with pattern to detect use-after-free
            std::memset(ptr, 0xDD, it->second.size);
        }
        
        TRACKED_FREE(ptr);
    }
    
    static bool checkPointer(void* ptr, const std::string& location) {
        auto it = guardedPointers.find(ptr);
        if (it != guardedPointers.end() && it->second.isFreed) {
            std::cout << "🚨 Use-after-free access detected!" << std::endl;
            std::cout << "  Pointer: " << ptr << std::endl;
            std::cout << "  Freed at: " << it->second.location << std::endl;
            std::cout << "  Access attempt at: " << location << std::endl;
            return false;
        }
        return true;
    }
    
    static void demonstrateUseAfterFreeDetection() {
        std::cout << "\n=== Use-After-Free Detection Demo ===" << std::endl;
        
        int* ptr = static_cast<int*>(allocateGuarded(sizeof(int), "main:allocation"));
        *ptr = 42;
        
        std::cout << "Allocated and initialized pointer: " << ptr << " = " << *ptr << std::endl;
        
        freeGuarded(ptr, "main:free");
        std::cout << "Pointer freed" << std::endl;
        
        // Attempt to use after free
        if (checkPointer(ptr, "main:use_after_free")) {
            std::cout << "Safe to use pointer" << std::endl;
        } else {
            std::cout << "✓ Use-after-free detected and prevented" << std::endl;
        }
        
        // Attempt double free
        freeGuarded(ptr, "main:double_free");
    }
};

std::map<void*, UseAfterFreeDetector::GuardedPointer> UseAfterFreeDetector::guardedPointers;

void demonstrateAdvancedDebugging() {
    std::cout << "\n=== Advanced Memory Debugging Demo ===" << std::endl;
    
    DebuggingAllocator::enableTracking();
    DebuggingAllocator::reset();
    
    // Test memory leak detection
    std::cout << "\n--- Memory Leak Test ---" << std::endl;
    void* ptr1 = TRACKED_MALLOC(100);
    void* ptr2 = TRACKED_MALLOC(200);
    void* ptr3 = TRACKED_MALLOC(50);
    
    TRACKED_FREE(ptr1);
    // Intentionally not freeing ptr2 and ptr3 to create leaks
    
    DebuggingAllocator::printStatistics();
    DebuggingAllocator::checkForLeaks();
    
    // Test stack overflow detection
    std::cout << "\n--- Stack Overflow Test ---" << std::endl;
    StackAnalyzer::recursiveFunction(5);  // Safe recursion
    // StackAnalyzer::recursiveFunction(2000);  // Would trigger safety check
    
    // Test buffer overflow detection
    BufferOverflowDetector::demonstrateBufferProtection();
    
    // Test use-after-free detection
    UseAfterFreeDetector::demonstrateUseAfterFreeDetection();
}

int main() {
    std::cout << "=== Memory Error Detection and Debugging Tools ===" << std::endl;
    
    // Run basic access violation demos
    // (Previous demos from AccessViolationDemo would go here)
    
    // Run advanced debugging demonstrations
    demonstrateAdvancedDebugging();
    
    std::cout << "\n=== Memory Safety Best Practices ===" << std::endl;
    std::cout << "✓ Always check pointers for null before dereferencing" << std::endl;
    std::cout << "✓ Use smart pointers (unique_ptr, shared_ptr) instead of raw pointers" << std::endl;
    std::cout << "✓ Prefer STL containers over raw arrays" << std::endl;
    std::cout << "✓ Use string classes instead of char arrays" << std::endl;
    std::cout << "✓ Enable compiler warnings and static analysis tools" << std::endl;
    std::cout << "✓ Use memory debugging tools (Valgrind, AddressSanitizer, etc.)" << std::endl;
    std::cout << "✓ Initialize pointers to nullptr" << std::endl;
    std::cout << "✓ Set pointers to nullptr after delete" << std::endl;
    std::cout << "✓ Use bounds-checking functions (at() instead of [])" << std::endl;
    std::cout << "✓ Follow RAII principles for resource management" << std::endl;
    
    return 0;
}
```

## Prevention Strategies

### Defensive Programming Techniques

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <algorithm>
#include <stdexcept>
#include <cassert>
#include <type_traits>

// Safe pointer wrapper
template<typename T>
class SafePointer {
private:
    T* ptr;
    bool isOwner;
    
public:
    SafePointer() : ptr(nullptr), isOwner(false) {}
    
    explicit SafePointer(T* p, bool owner = false) : ptr(p), isOwner(owner) {}
    
    SafePointer(const SafePointer&) = delete;  // Prevent copying
    SafePointer& operator=(const SafePointer&) = delete;
    
    SafePointer(SafePointer&& other) noexcept : ptr(other.ptr), isOwner(other.isOwner) {
        other.ptr = nullptr;
        other.isOwner = false;
    }
    
    SafePointer& operator=(SafePointer&& other) noexcept {
        if (this != &other) {
            reset();
            ptr = other.ptr;
            isOwner = other.isOwner;
            other.ptr = nullptr;
            other.isOwner = false;
        }
        return *this;
    }
    
    ~SafePointer() {
        reset();
    }
    
    T& operator*() const {
        if (!ptr) {
            throw std::runtime_error("Attempt to dereference null SafePointer");
        }
        return *ptr;
    }
    
    T* operator->() const {
        if (!ptr) {
            throw std::runtime_error("Attempt to access null SafePointer");
        }
        return ptr;
    }
    
    T* get() const { return ptr; }
    
    bool isValid() const { return ptr != nullptr; }
    
    void reset(T* newPtr = nullptr, bool owner = false) {
        if (isOwner && ptr) {
            delete ptr;
        }
        ptr = newPtr;
        isOwner = owner;
    }
    
    T* release() {
        T* temp = ptr;
        ptr = nullptr;
        isOwner = false;
        return temp;
    }
};

// Safe array with bounds checking
template<typename T, size_t SIZE>
class SafeArray {
private:
    T data[SIZE];
    
public:
    SafeArray() {
        // Initialize with default values
        for (size_t i = 0; i < SIZE; ++i) {
            data[i] = T{};
        }
    }
    
    T& at(size_t index) {
        if (index >= SIZE) {
            throw std::out_of_range("SafeArray index " + std::to_string(index) + 
                                  " out of range [0, " + std::to_string(SIZE-1) + "]");
        }
        return data[index];
    }
    
    const T& at(size_t index) const {
        if (index >= SIZE) {
            throw std::out_of_range("SafeArray index " + std::to_string(index) + 
                                  " out of range [0, " + std::to_string(SIZE-1) + "]");
        }
        return data[index];
    }
    
    T& operator[](size_t index) {
        assert(index < SIZE && "Array index out of bounds");
        return data[index];
    }
    
    const T& operator[](size_t index) const {
        assert(index < SIZE && "Array index out of bounds");
        return data[index];
    }
    
    size_t size() const { return SIZE; }
    
    T* getData() { return data; }
    const T* getData() const { return data; }
    
    // Iterator support
    T* begin() { return data; }
    T* end() { return data + SIZE; }
    const T* begin() const { return data; }
    const T* end() const { return data + SIZE; }
    const T* cbegin() const { return data; }
    const T* cend() const { return data + SIZE; }
};

// Safe string operations
class SafeString {
private:
    std::string str;
    
public:
    SafeString() = default;
    SafeString(const char* cstr) : str(cstr ? cstr : "") {}
    SafeString(const std::string& s) : str(s) {}
    
    // Safe substring operation
    SafeString substr(size_t pos, size_t len = std::string::npos) const {
        if (pos > str.length()) {
            throw std::out_of_range("SafeString substr position " + std::to_string(pos) + 
                                  " out of range [0, " + std::to_string(str.length()) + "]");
        }
        return SafeString(str.substr(pos, len));
    }
    
    // Safe character access
    char at(size_t index) const {
        if (index >= str.length()) {
            throw std::out_of_range("SafeString index " + std::to_string(index) + 
                                  " out of range [0, " + std::to_string(str.length()-1) + "]");
        }
        return str[index];
    }
    
    // Safe concatenation with size limits
    SafeString& safeConcatenate(const SafeString& other, size_t maxLength = 1000) {
        if (str.length() + other.str.length() > maxLength) {
            throw std::length_error("SafeString concatenation would exceed maximum length " + 
                                  std::to_string(maxLength));
        }
        str += other.str;
        return *this;
    }
    
    // Safe copy to C-style string
    void copyToBuffer(char* buffer, size_t bufferSize) const {
        if (!buffer) {
            throw std::invalid_argument("Buffer cannot be null");
        }
        if (bufferSize == 0) {
            throw std::invalid_argument("Buffer size cannot be zero");
        }
        
        size_t copyLength = std::min(str.length(), bufferSize - 1);
        str.copy(buffer, copyLength);
        buffer[copyLength] = '\0';
    }
    
    size_t length() const { return str.length(); }
    bool empty() const { return str.empty(); }
    const std::string& getString() const { return str; }
    
    friend std::ostream& operator<<(std::ostream& os, const SafeString& ss) {
        return os << ss.str;
    }
};

// Memory-safe dynamic array
template<typename T>
class SafeDynamicArray {
private:
    std::unique_ptr<T[]> data;
    size_t arraySize;
    size_t capacity;
    
    void resize(size_t newCapacity) {
        if (newCapacity <= capacity) return;
        
        auto newData = std::make_unique<T[]>(newCapacity);
        
        // Copy existing data
        for (size_t i = 0; i < arraySize; ++i) {
            newData[i] = std::move(data[i]);
        }
        
        data = std::move(newData);
        capacity = newCapacity;
    }
    
public:
    SafeDynamicArray(size_t initialCapacity = 10) 
        : data(std::make_unique<T[]>(initialCapacity)), arraySize(0), capacity(initialCapacity) {}
    
    void push_back(const T& value) {
        if (arraySize >= capacity) {
            resize(capacity * 2);
        }
        data[arraySize++] = value;
    }
    
    void push_back(T&& value) {
        if (arraySize >= capacity) {
            resize(capacity * 2);
        }
        data[arraySize++] = std::move(value);
    }
    
    T& at(size_t index) {
        if (index >= arraySize) {
            throw std::out_of_range("SafeDynamicArray index " + std::to_string(index) + 
                                  " out of range [0, " + std::to_string(arraySize-1) + "]");
        }
        return data[index];
    }
    
    const T& at(size_t index) const {
        if (index >= arraySize) {
            throw std::out_of_range("SafeDynamicArray index " + std::to_string(index) + 
                                  " out of range [0, " + std::to_string(arraySize-1) + "]");
        }
        return data[index];
    }
    
    size_t size() const { return arraySize; }
    size_t getCapacity() const { return capacity; }
    bool empty() const { return arraySize == 0; }
    
    // Safe iteration
    class Iterator {
    private:
        T* ptr;
        T* end;
        
    public:
        Iterator(T* p, T* e) : ptr(p), end(e) {}
        
        T& operator*() {
            if (ptr >= end) {
                throw std::out_of_range("Iterator out of range");
            }
            return *ptr;
        }
        
        Iterator& operator++() {
            ++ptr;
            return *this;
        }
        
        bool operator!=(const Iterator& other) const {
            return ptr != other.ptr;
        }
    };
    
    Iterator begin() { return Iterator(data.get(), data.get() + arraySize); }
    Iterator end() { return Iterator(data.get() + arraySize, data.get() + arraySize); }
};

// Input validation utilities
class InputValidator {
public:
    static bool isValidEmail(const std::string& email) {
        // Basic email validation
        size_t atPos = email.find('@');
        if (atPos == std::string::npos || atPos == 0 || atPos == email.length() - 1) {
            return false;
        }
        
        size_t dotPos = email.find('.', atPos);
        return dotPos != std::string::npos && dotPos < email.length() - 1;
    }
    
    static bool isValidInteger(const std::string& str, int& result) {
        if (str.empty()) return false;
        
        try {
            size_t processed;
            result = std::stoi(str, &processed);
            return processed == str.length();
        } catch (const std::exception&) {
            return false;
        }
    }
    
    static bool isValidRange(int value, int min, int max) {
        return value >= min && value <= max;
    }
    
    static std::string sanitizeInput(const std::string& input, size_t maxLength = 100) {
        std::string sanitized = input;
        
        // Truncate if too long
        if (sanitized.length() > maxLength) {
            sanitized = sanitized.substr(0, maxLength);
        }
        
        // Remove dangerous characters
        sanitized.erase(std::remove_if(sanitized.begin(), sanitized.end(),
            [](char c) { return c == '\0' || c < 32 || c > 126; }), sanitized.end());
        
        return sanitized;
    }
};

// Demonstration of defensive programming
void demonstrateDefensiveProgramming() {
    std::cout << "\n=== Defensive Programming Demo ===" << std::endl;
    
    // Safe pointer usage
    std::cout << "\n--- Safe Pointer Demo ---" << std::endl;
    try {
        SafePointer<int> ptr1;
        std::cout << "Empty pointer valid: " << ptr1.isValid() << std::endl;
        
        SafePointer<int> ptr2(new int(42), true);  // Takes ownership
        std::cout << "Owned pointer value: " << *ptr2 << std::endl;
        
        // ptr2 will automatically delete the memory when destroyed
        
    } catch (const std::exception& e) {
        std::cout << "Exception: " << e.what() << std::endl;
    }
    
    // Safe array usage
    std::cout << "\n--- Safe Array Demo ---" << std::endl;
    try {
        SafeArray<int, 5> arr;
        for (size_t i = 0; i < arr.size(); ++i) {
            arr[i] = static_cast<int>(i * 10);
        }
        
        std::cout << "Array contents: ";
        for (size_t i = 0; i < arr.size(); ++i) {
            std::cout << arr.at(i) << " ";
        }
        std::cout << std::endl;
        
        // This will throw an exception
        std::cout << "Accessing out of bounds: ";
        std::cout << arr.at(10) << std::endl;
        
    } catch (const std::exception& e) {
        std::cout << "✓ Exception caught: " << e.what() << std::endl;
    }
    
    // Safe string usage
    std::cout << "\n--- Safe String Demo ---" << std::endl;
    try {
        SafeString str("Hello, World!");
        std::cout << "Original string: " << str << std::endl;
        
        SafeString sub = str.substr(7, 5);
        std::cout << "Substring: " << sub << std::endl;
        
        // Safe buffer copy
        char buffer[20];
        str.copyToBuffer(buffer, sizeof(buffer));
        std::cout << "Copied to buffer: " << buffer << std::endl;
        
        // This will throw an exception
        SafeString invalid = str.substr(100);
        
    } catch (const std::exception& e) {
        std::cout << "✓ Exception caught: " << e.what() << std::endl;
    }
    
    // Safe dynamic array usage
    std::cout << "\n--- Safe Dynamic Array Demo ---" << std::endl;
    try {
        SafeDynamicArray<int> dynArr;
        
        for (int i = 0; i < 15; ++i) {
            dynArr.push_back(i * i);
        }
        
        std::cout << "Dynamic array size: " << dynArr.size() << std::endl;
        std::cout << "Capacity: " << dynArr.getCapacity() << std::endl;
        
        std::cout << "Contents: ";
        for (size_t i = 0; i < dynArr.size(); ++i) {
            std::cout << dynArr.at(i) << " ";
        }
        std::cout << std::endl;
        
        // This will throw an exception
        std::cout << dynArr.at(20) << std::endl;
        
    } catch (const std::exception& e) {
        std::cout << "✓ Exception caught: " << e.what() << std::endl;
    }
    
    // Input validation demo
    std::cout << "\n--- Input Validation Demo ---" << std::endl;
    
    std::vector<std::string> emails = {
        "user@example.com",
        "invalid-email",
        "user@",
        "@domain.com",
        "user@domain.com"
    };
    
    for (const auto& email : emails) {
        bool valid = InputValidator::isValidEmail(email);
        std::cout << "Email '" << email << "' is " << (valid ? "valid" : "invalid") << std::endl;
    }
    
    std::vector<std::string> numbers = {"123", "abc", "456def", "-789", "0"};
    for (const auto& numStr : numbers) {
        int result;
        bool valid = InputValidator::isValidInteger(numStr, result);
        std::cout << "String '" << numStr << "' is " << (valid ? "valid integer: " + std::to_string(result) : "invalid integer") << std::endl;
    }
    
    std::string unsafeInput = "Hello\x00World\x01\x02\x03!";  // Contains null and control characters
    std::string safeInput = InputValidator::sanitizeInput(unsafeInput);
    std::cout << "Sanitized input: '" << safeInput << "'" << std::endl;
}

int main() {
    demonstrateDefensiveProgramming();
    
    std::cout << "\n=== Access Violation Prevention Best Practices ===" << std::endl;
    std::cout << "✓ Use safe wrapper classes for pointers and arrays" << std::endl;
    std::cout << "✓ Always validate input parameters" << std::endl;
    std::cout << "✓ Use exceptions for error handling instead of error codes" << std::endl;
    std::cout << "✓ Implement bounds checking for all array/container access" << std::endl;
    std::cout << "✓ Use RAII for automatic resource management" << std::endl;
    std::cout << "✓ Prefer smart pointers over raw pointers" << std::endl;
    std::cout << "✓ Use STL containers instead of raw arrays" << std::endl;
    std::cout << "✓ Enable compiler warnings and treat warnings as errors" << std::endl;
    std::cout << "✓ Use static analysis tools and sanitizers" << std::endl;
    std::cout << "✓ Write comprehensive unit tests including edge cases" << std::endl;
    
    return 0;
}
```

Erişim ihlalleri, C++ programcılarının karşılaştığı en yaygın ve tehlikeli hata türleridir. Bu hatalar çoğunlukla önlenebilir ve doğru yaklaşımlarla güvenli kod yazılabilir. Modern C++ features ve defensive programming teknikleri kullanarak memory safety sağlanabilir.
