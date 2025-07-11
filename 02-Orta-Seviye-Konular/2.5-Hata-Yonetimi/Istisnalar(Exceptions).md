# İstisnalar (Exceptions)

İstisnalar (exceptions), C++'da runtime hatalarını ele almanın modern ve güvenli yoludur. Exception handling mekanizması, programın normal akışını bozmadan hataları yakalayıp işlemenizi sağlar ve RAII (Resource Acquisition Is Initialization) prensibi ile birlikte güvenli kod yazmanın temelini oluşturur.

## Temel Exception Handling

### Try-Catch-Throw Mechanism

```cpp
#include <iostream>
#include <string>
#include <stdexcept>
#include <vector>
#include <memory>
#include <fstream>

// Basic exception handling demonstration
void basicExceptionDemo() {
    std::cout << "=== Basic Exception Handling Demo ===" << std::endl;
    
    // Example 1: Division by zero
    std::cout << "\n--- Division by Zero Example ---" << std::endl;
    try {
        int numerator = 100;
        int denominator = 0;
        
        if (denominator == 0) {
            throw std::runtime_error("Division by zero attempted!");
        }
        
        int result = numerator / denominator;
        std::cout << "Result: " << result << std::endl;
        
    } catch (const std::runtime_error& e) {
        std::cout << "Caught runtime_error: " << e.what() << std::endl;
    }
    
    // Example 2: Array bounds checking
    std::cout << "\n--- Array Bounds Example ---" << std::endl;
    try {
        std::vector<int> numbers = {1, 2, 3, 4, 5};
        
        std::cout << "Accessing valid index (2): " << numbers.at(2) << std::endl;
        
        // This will throw std::out_of_range
        std::cout << "Accessing invalid index (10): " << numbers.at(10) << std::endl;
        
    } catch (const std::out_of_range& e) {
        std::cout << "Caught out_of_range: " << e.what() << std::endl;
    }
    
    // Example 3: Multiple catch blocks
    std::cout << "\n--- Multiple Catch Blocks Example ---" << std::endl;
    try {
        int choice;
        std::cout << "Enter choice (1=bad_alloc, 2=runtime_error, 3=logic_error): ";
        std::cin >> choice;
        
        switch (choice) {
            case 1:
                throw std::bad_alloc();
            case 2:
                throw std::runtime_error("Runtime error occurred");
            case 3:
                throw std::logic_error("Logic error occurred");
            default:
                throw std::invalid_argument("Invalid choice");
        }
        
    } catch (const std::bad_alloc& e) {
        std::cout << "Memory allocation failed: " << e.what() << std::endl;
    } catch (const std::runtime_error& e) {
        std::cout << "Runtime error: " << e.what() << std::endl;
    } catch (const std::logic_error& e) {
        std::cout << "Logic error: " << e.what() << std::endl;
    } catch (const std::exception& e) {
        std::cout << "General exception: " << e.what() << std::endl;
    } catch (...) {
        std::cout << "Unknown exception caught!" << std::endl;
    }
    
    std::cout << "\nProgram continues after exception handling..." << std::endl;
}

// Function that demonstrates exception propagation
int riskyFunction(int value) {
    std::cout << "riskyFunction called with value: " << value << std::endl;
    
    if (value < 0) {
        throw std::invalid_argument("Negative values not allowed");
    }
    if (value == 0) {
        throw std::runtime_error("Zero value causes runtime error");
    }
    if (value > 100) {
        throw std::out_of_range("Value too large");
    }
    
    return value * 2;
}

void intermediateFunction(int value) {
    std::cout << "intermediateFunction called" << std::endl;
    
    // This function doesn't catch exceptions - they propagate up
    int result = riskyFunction(value);
    std::cout << "Result from riskyFunction: " << result << std::endl;
}

void demonstrateExceptionPropagation() {
    std::cout << "\n=== Exception Propagation Demo ===" << std::endl;
    
    std::vector<int> testValues = {-5, 0, 50, 150};
    
    for (int value : testValues) {
        std::cout << "\n--- Testing value: " << value << " ---" << std::endl;
        try {
            intermediateFunction(value);
            std::cout << "Successfully processed value: " << value << std::endl;
            
        } catch (const std::invalid_argument& e) {
            std::cout << "Invalid argument caught: " << e.what() << std::endl;
        } catch (const std::runtime_error& e) {
            std::cout << "Runtime error caught: " << e.what() << std::endl;
        } catch (const std::out_of_range& e) {
            std::cout << "Out of range caught: " << e.what() << std::endl;
        }
    }
}

int main() {
    basicExceptionDemo();
    demonstrateExceptionPropagation();
    return 0;
}
```

## Custom Exception Classes

### User-Defined Exceptions

```cpp
#include <iostream>
#include <string>
#include <stdexcept>
#include <vector>
#include <memory>
#include <sstream>
#include <chrono>
#include <ctime>

// Base custom exception class
class ApplicationException : public std::exception {
protected:
    std::string message;
    std::string timestamp;
    std::string location;
    
public:
    ApplicationException(const std::string& msg, 
                        const std::string& loc = "Unknown") 
        : message(msg), location(loc) {
        // Add timestamp
        auto now = std::chrono::system_clock::now();
        auto time_t = std::chrono::system_clock::to_time_t(now);
        auto tm = *std::localtime(&time_t);
        
        std::ostringstream oss;
        oss << std::put_time(&tm, "%Y-%m-%d %H:%M:%S");
        timestamp = oss.str();
    }
    
    const char* what() const noexcept override {
        static std::string fullMessage;
        fullMessage = "[" + timestamp + "] " + message + " (Location: " + location + ")";
        return fullMessage.c_str();
    }
    
    const std::string& getMessage() const { return message; }
    const std::string& getTimestamp() const { return timestamp; }
    const std::string& getLocation() const { return location; }
};

// Specific exception types
class DatabaseException : public ApplicationException {
private:
    int errorCode;
    std::string query;
    
public:
    DatabaseException(const std::string& msg, int code, 
                     const std::string& sql = "", 
                     const std::string& loc = "Database") 
        : ApplicationException(msg, loc), errorCode(code), query(sql) {}
    
    int getErrorCode() const { return errorCode; }
    const std::string& getQuery() const { return query; }
    
    const char* what() const noexcept override {
        static std::string fullMessage;
        std::ostringstream oss;
        oss << "[" << timestamp << "] Database Error " << errorCode 
            << ": " << message << " (Location: " << location << ")";
        if (!query.empty()) {
            oss << " Query: '" << query << "'";
        }
        fullMessage = oss.str();
        return fullMessage.c_str();
    }
};

class NetworkException : public ApplicationException {
private:
    std::string serverAddress;
    int responseCode;
    
public:
    NetworkException(const std::string& msg, const std::string& server, 
                    int code = -1, const std::string& loc = "Network") 
        : ApplicationException(msg, loc), serverAddress(server), responseCode(code) {}
    
    const std::string& getServerAddress() const { return serverAddress; }
    int getResponseCode() const { return responseCode; }
    
    const char* what() const noexcept override {
        static std::string fullMessage;
        std::ostringstream oss;
        oss << "[" << timestamp << "] Network Error: " << message 
            << " (Server: " << serverAddress;
        if (responseCode != -1) {
            oss << ", Response Code: " << responseCode;
        }
        oss << ", Location: " << location << ")";
        fullMessage = oss.str();
        return fullMessage.c_str();
    }
};

class ValidationException : public ApplicationException {
private:
    std::string fieldName;
    std::string actualValue;
    std::string expectedFormat;
    
public:
    ValidationException(const std::string& msg, const std::string& field,
                       const std::string& actual = "", 
                       const std::string& expected = "",
                       const std::string& loc = "Validation") 
        : ApplicationException(msg, loc), fieldName(field), 
          actualValue(actual), expectedFormat(expected) {}
    
    const std::string& getFieldName() const { return fieldName; }
    const std::string& getActualValue() const { return actualValue; }
    const std::string& getExpectedFormat() const { return expectedFormat; }
    
    const char* what() const noexcept override {
        static std::string fullMessage;
        std::ostringstream oss;
        oss << "[" << timestamp << "] Validation Error: " << message 
            << " (Field: " << fieldName;
        if (!actualValue.empty()) {
            oss << ", Actual: '" << actualValue << "'";
        }
        if (!expectedFormat.empty()) {
            oss << ", Expected: " << expectedFormat;
        }
        oss << ", Location: " << location << ")";
        fullMessage = oss.str();
        return fullMessage.c_str();
    }
};

// Example classes that use custom exceptions
class User {
private:
    std::string username;
    std::string email;
    int age;
    
public:
    User(const std::string& name, const std::string& mail, int userAge) {
        setUsername(name);
        setEmail(mail);
        setAge(userAge);
    }
    
    void setUsername(const std::string& name) {
        if (name.empty()) {
            throw ValidationException("Username cannot be empty", "username", 
                                    name, "non-empty string", "User::setUsername");
        }
        if (name.length() < 3) {
            throw ValidationException("Username too short", "username", 
                                    name, "at least 3 characters", "User::setUsername");
        }
        if (name.length() > 20) {
            throw ValidationException("Username too long", "username", 
                                    name, "maximum 20 characters", "User::setUsername");
        }
        username = name;
    }
    
    void setEmail(const std::string& mail) {
        if (mail.empty()) {
            throw ValidationException("Email cannot be empty", "email", 
                                    mail, "valid email format", "User::setEmail");
        }
        if (mail.find('@') == std::string::npos) {
            throw ValidationException("Invalid email format", "email", 
                                    mail, "format: user@domain.com", "User::setEmail");
        }
        email = mail;
    }
    
    void setAge(int userAge) {
        if (userAge < 0) {
            throw ValidationException("Age cannot be negative", "age", 
                                    std::to_string(userAge), "positive integer", "User::setAge");
        }
        if (userAge > 150) {
            throw ValidationException("Age too high", "age", 
                                    std::to_string(userAge), "realistic age", "User::setAge");
        }
        age = userAge;
    }
    
    const std::string& getUsername() const { return username; }
    const std::string& getEmail() const { return email; }
    int getAge() const { return age; }
    
    void displayInfo() const {
        std::cout << "User: " << username << ", Email: " << email << ", Age: " << age << std::endl;
    }
};

class DatabaseManager {
public:
    void connect(const std::string& connectionString) {
        if (connectionString.empty()) {
            throw DatabaseException("Connection string is empty", 1001, "", "DatabaseManager::connect");
        }
        
        // Simulate connection failure
        if (connectionString.find("localhost") == std::string::npos) {
            throw DatabaseException("Cannot connect to remote server", 1002, "", "DatabaseManager::connect");
        }
        
        std::cout << "Connected to database: " << connectionString << std::endl;
    }
    
    std::vector<User> executeQuery(const std::string& query) {
        if (query.empty()) {
            throw DatabaseException("Query cannot be empty", 2001, query, "DatabaseManager::executeQuery");
        }
        
        if (query.find("DROP") != std::string::npos) {
            throw DatabaseException("DROP statements not allowed", 2002, query, "DatabaseManager::executeQuery");
        }
        
        if (query.find("SELECT") == std::string::npos) {
            throw DatabaseException("Only SELECT statements supported", 2003, query, "DatabaseManager::executeQuery");
        }
        
        std::cout << "Executing query: " << query << std::endl;
        
        // Simulate query results
        std::vector<User> results;
        try {
            results.emplace_back("john_doe", "john@example.com", 25);
            results.emplace_back("jane_smith", "jane@example.com", 30);
        } catch (const ValidationException& e) {
            throw DatabaseException("Error creating user from query results", 2004, query, "DatabaseManager::executeQuery");
        }
        
        return results;
    }
};

class NetworkClient {
public:
    std::string sendRequest(const std::string& server, const std::string& endpoint) {
        if (server.empty()) {
            throw NetworkException("Server address cannot be empty", server, -1, "NetworkClient::sendRequest");
        }
        
        // Simulate different network scenarios
        if (server == "unreachable.server.com") {
            throw NetworkException("Server unreachable", server, -1, "NetworkClient::sendRequest");
        }
        
        if (server == "error.server.com") {
            throw NetworkException("Server returned error", server, 500, "NetworkClient::sendRequest");
        }
        
        if (endpoint.find("/api/") == std::string::npos) {
            throw NetworkException("Invalid API endpoint", server, 404, "NetworkClient::sendRequest");
        }
        
        std::cout << "Successfully sent request to " << server << endpoint << std::endl;
        return "Response data from " + server;
    }
};

void demonstrateCustomExceptions() {
    std::cout << "\n=== Custom Exceptions Demo ===" << std::endl;
    
    // Test User validation
    std::cout << "\n--- User Validation Tests ---" << std::endl;
    std::vector<std::tuple<std::string, std::string, int>> userData = {
        {"john_doe", "john@example.com", 25},  // Valid
        {"", "empty@example.com", 30},         // Empty username
        {"ab", "short@example.com", 25},       // Username too short
        {"valid_user", "invalid_email", 28},   // Invalid email
        {"negative_age", "user@example.com", -5}, // Negative age
        {"old_user", "old@example.com", 200}   // Age too high
    };
    
    for (const auto& [name, email, age] : userData) {
        try {
            User user(name, email, age);
            std::cout << "✓ User created successfully: ";
            user.displayInfo();
        } catch (const ValidationException& e) {
            std::cout << "✗ Validation failed: " << e.what() << std::endl;
            std::cout << "  Field: " << e.getFieldName() 
                      << ", Actual: '" << e.getActualValue() 
                      << "', Expected: " << e.getExpectedFormat() << std::endl;
        }
    }
    
    // Test Database operations
    std::cout << "\n--- Database Operations Tests ---" << std::endl;
    DatabaseManager db;
    
    std::vector<std::string> connections = {
        "localhost:3306/mydb",     // Valid
        "",                        // Empty
        "remote.server.com:3306"   // Remote (invalid)
    };
    
    for (const auto& conn : connections) {
        try {
            db.connect(conn);
            std::cout << "✓ Database connected successfully" << std::endl;
            
            // Test queries
            std::vector<std::string> queries = {
                "SELECT * FROM users",           // Valid
                "",                             // Empty
                "DROP TABLE users",             // Not allowed
                "INSERT INTO users VALUES (1)"  // Not SELECT
            };
            
            for (const auto& query : queries) {
                try {
                    auto results = db.executeQuery(query);
                    std::cout << "✓ Query executed successfully, " << results.size() << " results" << std::endl;
                } catch (const DatabaseException& e) {
                    std::cout << "✗ Database query failed: " << e.what() << std::endl;
                    std::cout << "  Error Code: " << e.getErrorCode() 
                              << ", Query: '" << e.getQuery() << "'" << std::endl;
                }
            }
            
        } catch (const DatabaseException& e) {
            std::cout << "✗ Database connection failed: " << e.what() << std::endl;
            std::cout << "  Error Code: " << e.getErrorCode() << std::endl;
        }
    }
    
    // Test Network operations
    std::cout << "\n--- Network Operations Tests ---" << std::endl;
    NetworkClient client;
    
    std::vector<std::pair<std::string, std::string>> requests = {
        {"api.example.com", "/api/users"},        // Valid
        {"", "/api/data"},                        // Empty server
        {"unreachable.server.com", "/api/test"},  // Unreachable
        {"error.server.com", "/api/data"},        // Server error
        {"valid.server.com", "/invalid/endpoint"} // Invalid endpoint
    };
    
    for (const auto& [server, endpoint] : requests) {
        try {
            std::string response = client.sendRequest(server, endpoint);
            std::cout << "✓ Network request successful: " << response << std::endl;
        } catch (const NetworkException& e) {
            std::cout << "✗ Network request failed: " << e.what() << std::endl;
            std::cout << "  Server: " << e.getServerAddress() 
                      << ", Response Code: " << e.getResponseCode() << std::endl;
        }
    }
}

int main() {
    basicExceptionDemo();
    demonstrateExceptionPropagation();
    demonstrateCustomExceptions();
    return 0;
}
```

## RAII ve Exception Safety

### Resource Management with Exceptions

```cpp
#include <iostream>
#include <string>
#include <memory>
#include <fstream>
#include <vector>
#include <mutex>
#include <thread>
#include <chrono>
#include <stdexcept>

// Non-RAII approach (problematic)
class UnsafeResourceManager {
private:
    FILE* file;
    int* dynamicArray;
    std::mutex* mtx;
    
public:
    UnsafeResourceManager(const std::string& filename, size_t arraySize) 
        : file(nullptr), dynamicArray(nullptr), mtx(nullptr) {
        
        std::cout << "UnsafeResourceManager: Acquiring resources..." << std::endl;
        
        // Acquire file resource
        file = fopen(filename.c_str(), "w");
        if (!file) {
            throw std::runtime_error("Failed to open file: " + filename);
        }
        
        // Acquire dynamic memory
        dynamicArray = new int[arraySize];
        
        // Acquire mutex
        mtx = new std::mutex();
        
        // Simulate potential failure after resource acquisition
        if (filename.find("fail") != std::string::npos) {
            throw std::runtime_error("Simulated failure after resource acquisition");
            // Memory leak! Resources not released
        }
        
        std::cout << "UnsafeResourceManager: All resources acquired successfully" << std::endl;
    }
    
    ~UnsafeResourceManager() {
        std::cout << "UnsafeResourceManager: Releasing resources..." << std::endl;
        cleanup();
    }
    
    void cleanup() {
        if (file) {
            fclose(file);
            file = nullptr;
            std::cout << "  File closed" << std::endl;
        }
        
        if (dynamicArray) {
            delete[] dynamicArray;
            dynamicArray = nullptr;
            std::cout << "  Dynamic array deleted" << std::endl;
        }
        
        if (mtx) {
            delete mtx;
            mtx = nullptr;
            std::cout << "  Mutex deleted" << std::endl;
        }
    }
    
    void doWork() {
        if (!file || !dynamicArray || !mtx) {
            throw std::runtime_error("Resources not properly initialized");
        }
        
        std::lock_guard<std::mutex> lock(*mtx);
        fprintf(file, "Working with resources...\n");
        dynamicArray[0] = 42;
        std::cout << "UnsafeResourceManager: Work completed" << std::endl;
    }
};

// RAII approach (safe)
class SafeResourceManager {
private:
    std::unique_ptr<std::ofstream> file;
    std::unique_ptr<int[]> dynamicArray;
    std::unique_ptr<std::mutex> mtx;
    std::string filename;
    
public:
    SafeResourceManager(const std::string& fname, size_t arraySize) 
        : filename(fname) {
        
        std::cout << "SafeResourceManager: Acquiring resources..." << std::endl;
        
        // Acquire file resource using RAII
        file = std::make_unique<std::ofstream>(filename);
        if (!file->is_open()) {
            throw std::runtime_error("Failed to open file: " + filename);
        }
        
        // Acquire dynamic memory using RAII
        dynamicArray = std::make_unique<int[]>(arraySize);
        
        // Acquire mutex using RAII
        mtx = std::make_unique<std::mutex>();
        
        // Simulate potential failure after resource acquisition
        if (filename.find("fail") != std::string::npos) {
            throw std::runtime_error("Simulated failure after resource acquisition");
            // No memory leak! RAII automatically cleans up
        }
        
        std::cout << "SafeResourceManager: All resources acquired successfully" << std::endl;
    }
    
    ~SafeResourceManager() {
        std::cout << "SafeResourceManager: Destructor called (RAII cleanup)" << std::endl;
        // RAII automatically handles cleanup
    }
    
    void doWork() {
        if (!file || !file->is_open() || !dynamicArray || !mtx) {
            throw std::runtime_error("Resources not properly initialized");
        }
        
        std::lock_guard<std::mutex> lock(*mtx);
        *file << "Working with resources safely...\n";
        dynamicArray[0] = 42;
        std::cout << "SafeResourceManager: Work completed" << std::endl;
    }
};

// Exception-safe function with multiple resources
class DatabaseConnection {
private:
    std::string connectionString;
    bool connected;
    
public:
    DatabaseConnection(const std::string& connStr) 
        : connectionString(connStr), connected(false) {
        std::cout << "Connecting to database: " << connectionString << std::endl;
        
        if (connStr.empty()) {
            throw std::invalid_argument("Connection string cannot be empty");
        }
        
        // Simulate connection
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        connected = true;
        std::cout << "Database connected successfully" << std::endl;
    }
    
    ~DatabaseConnection() {
        if (connected) {
            std::cout << "Disconnecting from database: " << connectionString << std::endl;
            connected = false;
        }
    }
    
    void executeQuery(const std::string& query) {
        if (!connected) {
            throw std::runtime_error("Not connected to database");
        }
        
        if (query.empty()) {
            throw std::invalid_argument("Query cannot be empty");
        }
        
        std::cout << "Executing query: " << query << std::endl;
        
        // Simulate query execution
        if (query.find("FAIL") != std::string::npos) {
            throw std::runtime_error("Query execution failed");
        }
        
        std::cout << "Query executed successfully" << std::endl;
    }
    
    bool isConnected() const { return connected; }
};

// RAII wrapper for multiple database operations
class DatabaseTransaction {
private:
    DatabaseConnection& db;
    bool committed;
    std::vector<std::string> operations;
    
public:
    DatabaseTransaction(DatabaseConnection& database) 
        : db(database), committed(false) {
        if (!db.isConnected()) {
            throw std::runtime_error("Database not connected");
        }
        std::cout << "Transaction started" << std::endl;
    }
    
    ~DatabaseTransaction() {
        if (!committed) {
            rollback();
        }
    }
    
    void addOperation(const std::string& operation) {
        if (committed) {
            throw std::runtime_error("Cannot add operations to committed transaction");
        }
        
        operations.push_back(operation);
        std::cout << "Operation added to transaction: " << operation << std::endl;
    }
    
    void execute() {
        if (committed) {
            throw std::runtime_error("Transaction already committed");
        }
        
        std::cout << "Executing transaction with " << operations.size() << " operations..." << std::endl;
        
        for (const auto& op : operations) {
            db.executeQuery(op);  // May throw exception
        }
        
        std::cout << "All operations executed successfully" << std::endl;
    }
    
    void commit() {
        if (committed) {
            throw std::runtime_error("Transaction already committed");
        }
        
        execute();  // Execute all operations first
        committed = true;
        std::cout << "Transaction committed successfully" << std::endl;
    }
    
    void rollback() {
        if (!committed) {
            std::cout << "Transaction rolled back (automatic cleanup)" << std::endl;
            operations.clear();
        }
    }
};

// Exception safety levels demonstration
class ExceptionSafetyDemo {
public:
    // Basic guarantee: No memory leaks, but object state may be changed
    static void basicGuaranteeExample() {
        std::cout << "\n--- Basic Guarantee Example ---" << std::endl;
        
        std::vector<int> data = {1, 2, 3, 4, 5};
        std::cout << "Original data: ";
        for (int val : data) std::cout << val << " ";
        std::cout << std::endl;
        
        try {
            // This operation may partially succeed before throwing
            data.reserve(1000000000);  // May throw bad_alloc
            data.push_back(6);
            data.push_back(7);
            
            // Simulate failure
            if (data.size() > 5) {
                throw std::runtime_error("Simulated failure");
            }
            
        } catch (const std::exception& e) {
            std::cout << "Exception caught: " << e.what() << std::endl;
            std::cout << "Data after exception: ";
            for (int val : data) std::cout << val << " ";
            std::cout << " (size: " << data.size() << ")" << std::endl;
        }
    }
    
    // Strong guarantee: Either succeeds completely or leaves object unchanged
    static void strongGuaranteeExample() {
        std::cout << "\n--- Strong Guarantee Example ---" << std::endl;
        
        std::vector<int> data = {1, 2, 3, 4, 5};
        std::cout << "Original data: ";
        for (int val : data) std::cout << val << " ";
        std::cout << std::endl;
        
        try {
            // Make a copy to work with (strong guarantee approach)
            std::vector<int> backup = data;
            
            // Perform operations on backup
            backup.push_back(6);
            backup.push_back(7);
            
            // Simulate potential failure
            if (backup.size() > 6) {
                throw std::runtime_error("Simulated failure during operation");
            }
            
            // Only commit changes if everything succeeded
            data = std::move(backup);
            std::cout << "Operation completed successfully" << std::endl;
            
        } catch (const std::exception& e) {
            std::cout << "Exception caught: " << e.what() << std::endl;
            std::cout << "Data remains unchanged: ";
            for (int val : data) std::cout << val << " ";
            std::cout << std::endl;
        }
    }
    
    // No-throw guarantee: Operations that never throw
    static void noThrowGuaranteeExample() noexcept {
        std::cout << "\n--- No-Throw Guarantee Example ---" << std::endl;
        
        // Operations that are guaranteed not to throw
        int a = 5, b = 10;
        int sum = a + b;  // No-throw
        
        std::cout << "No-throw operations: " << a << " + " << b << " = " << sum << std::endl;
        
        // Swap operations are typically no-throw
        std::cout << "Before swap: a = " << a << ", b = " << b << std::endl;
        std::swap(a, b);  // No-throw guarantee
        std::cout << "After swap: a = " << a << ", b = " << b << std::endl;
    }
};

void demonstrateRAIIandExceptionSafety() {
    std::cout << "\n=== RAII and Exception Safety Demo ===" << std::endl;
    
    // Test unsafe resource management
    std::cout << "\n--- Unsafe Resource Management ---" << std::endl;
    try {
        UnsafeResourceManager unsafe("test.txt", 100);
        unsafe.doWork();
        std::cout << "Unsafe manager: Normal operation completed" << std::endl;
    } catch (const std::exception& e) {
        std::cout << "Exception in unsafe manager: " << e.what() << std::endl;
    }
    
    std::cout << "\nTesting unsafe manager with failure:" << std::endl;
    try {
        UnsafeResourceManager unsafe("test_fail.txt", 100);
        // This will throw after acquiring some resources
    } catch (const std::exception& e) {
        std::cout << "Exception in unsafe manager: " << e.what() << std::endl;
        std::cout << "Note: Resources may have leaked!" << std::endl;
    }
    
    // Test safe resource management with RAII
    std::cout << "\n--- Safe Resource Management (RAII) ---" << std::endl;
    try {
        SafeResourceManager safe("test_safe.txt", 100);
        safe.doWork();
        std::cout << "Safe manager: Normal operation completed" << std::endl;
    } catch (const std::exception& e) {
        std::cout << "Exception in safe manager: " << e.what() << std::endl;
    }
    
    std::cout << "\nTesting safe manager with failure:" << std::endl;
    try {
        SafeResourceManager safe("test_safe_fail.txt", 100);
        // This will throw after acquiring some resources, but RAII cleans up
    } catch (const std::exception& e) {
        std::cout << "Exception in safe manager: " << e.what() << std::endl;
        std::cout << "Note: RAII ensured proper cleanup!" << std::endl;
    }
    
    // Test database transaction with RAII
    std::cout << "\n--- Database Transaction Example ---" << std::endl;
    try {
        DatabaseConnection db("localhost:5432/mydb");
        
        // Successful transaction
        {
            DatabaseTransaction tx(db);
            tx.addOperation("INSERT INTO users VALUES (1, 'John')");
            tx.addOperation("INSERT INTO users VALUES (2, 'Jane')");
            tx.commit();
        } // Transaction destructor called here
        
        // Failed transaction (automatic rollback)
        {
            DatabaseTransaction tx(db);
            tx.addOperation("INSERT INTO users VALUES (3, 'Bob')");
            tx.addOperation("FAIL QUERY");  // This will cause failure
            tx.commit();  // This will throw
        } // Transaction destructor called here (automatic rollback)
        
    } catch (const std::exception& e) {
        std::cout << "Database exception: " << e.what() << std::endl;
    }
    
    // Exception safety levels
    ExceptionSafetyDemo::basicGuaranteeExample();
    ExceptionSafetyDemo::strongGuaranteeExample();
    ExceptionSafetyDemo::noThrowGuaranteeExample();
}

int main() {
    basicExceptionDemo();
    demonstrateExceptionPropagation();
    demonstrateCustomExceptions();
    demonstrateRAIIandExceptionSafety();
    
    std::cout << "\n=== Exception Handling Best Practices ===" << std::endl;
    std::cout << "✓ Use RAII for automatic resource management" << std::endl;
    std::cout << "✓ Catch exceptions by const reference" << std::endl;
    std::cout << "✓ Create custom exception types for different error categories" << std::endl;
    std::cout << "✓ Provide strong exception safety guarantees when possible" << std::endl;
    std::cout << "✓ Use smart pointers instead of raw pointers" << std::endl;
    std::cout << "✓ Don't throw exceptions from destructors" << std::endl;
    std::cout << "✓ Handle exceptions at appropriate levels" << std::endl;
    std::cout << "✓ Log exception details for debugging" << std::endl;
    
    return 0;
}
```

İstisnalar, C++'da güvenli ve maintainable kod yazmanın anahtarıdır. RAII prensibi ile birlikte kullanıldığında, resource leaks'i önler ve exception safety garantileri sağlar.
