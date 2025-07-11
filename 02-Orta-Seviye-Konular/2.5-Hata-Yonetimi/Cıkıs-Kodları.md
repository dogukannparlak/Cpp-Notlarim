# Çıkış Kodları (Exit Codes)

Çıkış kodları, C++ programlarının terminasyon durumunu belirten sayısal değerlerdir. Bu kodlar operating system'e programın başarılı olup olmadığını, hangi tür hatayla karşılaştığını bildirmek için kullanılır ve system integration, automation ve debugging için kritik öneme sahiptir.

## Temel Exit Code Kavramları

### Standard Exit Codes ve Anlamları

```cpp
#include <iostream>
#include <cstdlib>
#include <string>
#include <fstream>
#include <vector>
#include <map>

// Standard exit codes
namespace ExitCodes {
    const int SUCCESS = 0;           // Program başarıyla tamamlandı
    const int GENERAL_ERROR = 1;     // Genel hata
    const int MISUSE = 2;           // Yanlış kullanım
    const int CANNOT_EXECUTE = 126;  // Komut çalıştırılamadı
    const int COMMAND_NOT_FOUND = 127; // Komut bulunamadı
    const int INVALID_EXIT = 128;    // Geçersiz exit argümanı
    const int FATAL_ERROR_SIGNAL = 128; // Fatal signal base (128 + signal number)
    
    // Custom application exit codes
    const int INVALID_ARGUMENTS = 10;
    const int FILE_NOT_FOUND = 11;
    const int PERMISSION_DENIED = 12;
    const int NETWORK_ERROR = 13;
    const int DATABASE_ERROR = 14;
    const int CONFIGURATION_ERROR = 15;
    const int MEMORY_ERROR = 16;
    const int TIMEOUT_ERROR = 17;
    const int USER_CANCELLED = 18;
    const int VALIDATION_ERROR = 19;
}

// Exit code information structure
struct ExitCodeInfo {
    int code;
    std::string name;
    std::string description;
    std::string category;
    
    ExitCodeInfo(int c, const std::string& n, const std::string& d, const std::string& cat)
        : code(c), name(n), description(d), category(cat) {}
};

class ExitCodeManager {
private:
    static std::map<int, ExitCodeInfo> exitCodeRegistry;
    static bool initialized;
    
    static void initializeRegistry() {
        if (initialized) return;
        
        // Standard codes
        exitCodeRegistry.emplace(ExitCodes::SUCCESS, 
            ExitCodeInfo(ExitCodes::SUCCESS, "SUCCESS", "Program completed successfully", "Standard"));
        exitCodeRegistry.emplace(ExitCodes::GENERAL_ERROR, 
            ExitCodeInfo(ExitCodes::GENERAL_ERROR, "GENERAL_ERROR", "General error", "Standard"));
        exitCodeRegistry.emplace(ExitCodes::MISUSE, 
            ExitCodeInfo(ExitCodes::MISUSE, "MISUSE", "Misuse of shell command", "Standard"));
        
        // Application-specific codes
        exitCodeRegistry.emplace(ExitCodes::INVALID_ARGUMENTS, 
            ExitCodeInfo(ExitCodes::INVALID_ARGUMENTS, "INVALID_ARGUMENTS", "Invalid command line arguments", "Application"));
        exitCodeRegistry.emplace(ExitCodes::FILE_NOT_FOUND, 
            ExitCodeInfo(ExitCodes::FILE_NOT_FOUND, "FILE_NOT_FOUND", "Required file not found", "File System"));
        exitCodeRegistry.emplace(ExitCodes::PERMISSION_DENIED, 
            ExitCodeInfo(ExitCodes::PERMISSION_DENIED, "PERMISSION_DENIED", "Permission denied", "Security"));
        exitCodeRegistry.emplace(ExitCodes::NETWORK_ERROR, 
            ExitCodeInfo(ExitCodes::NETWORK_ERROR, "NETWORK_ERROR", "Network communication error", "Network"));
        exitCodeRegistry.emplace(ExitCodes::DATABASE_ERROR, 
            ExitCodeInfo(ExitCodes::DATABASE_ERROR, "DATABASE_ERROR", "Database operation failed", "Database"));
        exitCodeRegistry.emplace(ExitCodes::CONFIGURATION_ERROR, 
            ExitCodeInfo(ExitCodes::CONFIGURATION_ERROR, "CONFIGURATION_ERROR", "Configuration file error", "Configuration"));
        exitCodeRegistry.emplace(ExitCodes::MEMORY_ERROR, 
            ExitCodeInfo(ExitCodes::MEMORY_ERROR, "MEMORY_ERROR", "Memory allocation failed", "System"));
        exitCodeRegistry.emplace(ExitCodes::TIMEOUT_ERROR, 
            ExitCodeInfo(ExitCodes::TIMEOUT_ERROR, "TIMEOUT_ERROR", "Operation timed out", "System"));
        exitCodeRegistry.emplace(ExitCodes::USER_CANCELLED, 
            ExitCodeInfo(ExitCodes::USER_CANCELLED, "USER_CANCELLED", "Operation cancelled by user", "User"));
        exitCodeRegistry.emplace(ExitCodes::VALIDATION_ERROR, 
            ExitCodeInfo(ExitCodes::VALIDATION_ERROR, "VALIDATION_ERROR", "Data validation failed", "Validation"));
        
        initialized = true;
    }
    
public:
    static void registerExitCode(int code, const std::string& name, 
                                const std::string& description, const std::string& category) {
        initializeRegistry();
        exitCodeRegistry.emplace(code, ExitCodeInfo(code, name, description, category));
    }
    
    static const ExitCodeInfo* getExitCodeInfo(int code) {
        initializeRegistry();
        auto it = exitCodeRegistry.find(code);
        return (it != exitCodeRegistry.end()) ? &it->second : nullptr;
    }
    
    static void printExitCodeInfo(int code) {
        const ExitCodeInfo* info = getExitCodeInfo(code);
        if (info) {
            std::cout << "Exit Code " << info->code << " (" << info->name << ")" << std::endl;
            std::cout << "  Description: " << info->description << std::endl;
            std::cout << "  Category: " << info->category << std::endl;
        } else {
            std::cout << "Exit Code " << code << " (UNKNOWN)" << std::endl;
            std::cout << "  Description: Unregistered exit code" << std::endl;
            std::cout << "  Category: Unknown" << std::endl;
        }
    }
    
    static void listAllExitCodes() {
        initializeRegistry();
        std::cout << "\n=== Registered Exit Codes ===" << std::endl;
        
        std::map<std::string, std::vector<const ExitCodeInfo*>> categories;
        for (const auto& pair : exitCodeRegistry) {
            categories[pair.second.category].push_back(&pair.second);
        }
        
        for (const auto& categoryPair : categories) {
            std::cout << "\n--- " << categoryPair.first << " ---" << std::endl;
            for (const ExitCodeInfo* info : categoryPair.second) {
                std::cout << "  " << info->code << " - " << info->name 
                          << ": " << info->description << std::endl;
            }
        }
    }
    
    [[noreturn]] static void exitWithCode(int code, const std::string& message = "") {
        if (!message.empty()) {
            std::cerr << "Exit Message: " << message << std::endl;
        }
        
        std::cerr << "Exiting with code: ";
        printExitCodeInfo(code);
        
        std::exit(code);
    }
};

std::map<int, ExitCodeInfo> ExitCodeManager::exitCodeRegistry;
bool ExitCodeManager::initialized = false;

// Example functions that demonstrate different exit scenarios
class FileProcessor {
public:
    static void processFile(const std::string& filename) {
        std::cout << "Processing file: " << filename << std::endl;
        
        // Check if file exists
        std::ifstream file(filename);
        if (!file.is_open()) {
            std::cerr << "Error: File '" << filename << "' not found" << std::endl;
            ExitCodeManager::exitWithCode(ExitCodes::FILE_NOT_FOUND, 
                                        "Cannot open file: " + filename);
        }
        
        // Simulate file processing
        std::string line;
        int lineCount = 0;
        while (std::getline(file, line)) {
            lineCount++;
            
            // Simulate processing error
            if (line.find("ERROR") != std::string::npos) {
                std::cerr << "Error processing line " << lineCount << ": " << line << std::endl;
                ExitCodeManager::exitWithCode(ExitCodes::GENERAL_ERROR, 
                                            "Processing error at line " + std::to_string(lineCount));
            }
        }
        
        std::cout << "Successfully processed " << lineCount << " lines" << std::endl;
    }
};

class NetworkClient {
public:
    static void connectToServer(const std::string& address, int port) {
        std::cout << "Connecting to " << address << ":" << port << std::endl;
        
        // Simulate different network scenarios
        if (address.empty()) {
            std::cerr << "Error: Server address cannot be empty" << std::endl;
            ExitCodeManager::exitWithCode(ExitCodes::INVALID_ARGUMENTS, 
                                        "Invalid server address");
        }
        
        if (port <= 0 || port > 65535) {
            std::cerr << "Error: Invalid port number: " << port << std::endl;
            ExitCodeManager::exitWithCode(ExitCodes::INVALID_ARGUMENTS, 
                                        "Port must be between 1 and 65535");
        }
        
        if (address == "unreachable.server.com") {
            std::cerr << "Error: Cannot reach server " << address << std::endl;
            ExitCodeManager::exitWithCode(ExitCodes::NETWORK_ERROR, 
                                        "Server unreachable: " + address);
        }
        
        if (address == "timeout.server.com") {
            std::cerr << "Error: Connection timeout to " << address << std::endl;
            ExitCodeManager::exitWithCode(ExitCodes::TIMEOUT_ERROR, 
                                        "Connection timeout");
        }
        
        std::cout << "Successfully connected to " << address << ":" << port << std::endl;
    }
};

class DatabaseManager {
public:
    static void initializeDatabase(const std::string& connectionString) {
        std::cout << "Initializing database with: " << connectionString << std::endl;
        
        if (connectionString.empty()) {
            std::cerr << "Error: Database connection string is empty" << std::endl;
            ExitCodeManager::exitWithCode(ExitCodes::CONFIGURATION_ERROR, 
                                        "Missing database configuration");
        }
        
        if (connectionString.find("invalid_db") != std::string::npos) {
            std::cerr << "Error: Invalid database configuration" << std::endl;
            ExitCodeManager::exitWithCode(ExitCodes::DATABASE_ERROR, 
                                        "Cannot connect to database");
        }
        
        std::cout << "Database initialized successfully" << std::endl;
    }
};

void demonstrateBasicExitCodes() {
    std::cout << "=== Basic Exit Codes Demonstration ===" << std::endl;
    
    // List all registered exit codes
    ExitCodeManager::listAllExitCodes();
    
    std::cout << "\n--- Exit Code Examples ---" << std::endl;
    
    // Example 1: Success case (this won't actually exit since we're demonstrating)
    std::cout << "\nExample 1: Success case" << std::endl;
    ExitCodeManager::printExitCodeInfo(ExitCodes::SUCCESS);
    
    // Example 2: Different error types
    std::cout << "\nExample 2: Error cases" << std::endl;
    std::vector<int> errorCodes = {
        ExitCodes::INVALID_ARGUMENTS,
        ExitCodes::FILE_NOT_FOUND,
        ExitCodes::NETWORK_ERROR,
        ExitCodes::DATABASE_ERROR,
        ExitCodes::TIMEOUT_ERROR
    };
    
    for (int code : errorCodes) {
        ExitCodeManager::printExitCodeInfo(code);
        std::cout << std::endl;
    }
}

int main(int argc, char* argv[]) {
    // Register any additional custom exit codes
    ExitCodeManager::registerExitCode(50, "CUSTOM_ERROR", "Application-specific error", "Custom");
    
    demonstrateBasicExitCodes();
    
    // Note: In a real application, these would actually call exit()
    // For demonstration, we'll show what would happen
    
    std::cout << "\n=== Simulated Exit Scenarios ===\n" << std::endl;
    std::cout << "Note: These are simulations - actual exit() calls are commented out\n" << std::endl;
    
    // Test different scenarios (commented out to prevent actual exit)
    /*
    if (argc < 2) {
        ExitCodeManager::exitWithCode(ExitCodes::INVALID_ARGUMENTS, 
                                    "Usage: program <command> [arguments]");
    }
    
    std::string command = argv[1];
    
    if (command == "process_file") {
        if (argc < 3) {
            ExitCodeManager::exitWithCode(ExitCodes::INVALID_ARGUMENTS, 
                                        "File path required");
        }
        FileProcessor::processFile(argv[2]);
        
    } else if (command == "connect") {
        if (argc < 4) {
            ExitCodeManager::exitWithCode(ExitCodes::INVALID_ARGUMENTS, 
                                        "Server address and port required");
        }
        NetworkClient::connectToServer(argv[2], std::atoi(argv[3]));
        
    } else if (command == "init_db") {
        if (argc < 3) {
            ExitCodeManager::exitWithCode(ExitCodes::INVALID_ARGUMENTS, 
                                        "Database connection string required");
        }
        DatabaseManager::initializeDatabase(argv[2]);
        
    } else {
        ExitCodeManager::exitWithCode(ExitCodes::INVALID_ARGUMENTS, 
                                    "Unknown command: " + command);
    }
    */
    
    // Simulate successful completion
    std::cout << "Program completed successfully" << std::endl;
    ExitCodeManager::printExitCodeInfo(ExitCodes::SUCCESS);
    
    return ExitCodes::SUCCESS;
}
```

## System Integration ve Process Management

### Advanced Exit Code Handling

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <memory>
#include <fstream>
#include <sstream>
#include <chrono>
#include <thread>
#include <cstdlib>
#include <csignal>

#ifdef _WIN32
    #include <windows.h>
    #include <process.h>
#else
    #include <unistd.h>
    #include <sys/wait.h>
    #include <sys/types.h>
#endif

// Enhanced exit code system with logging and process management
class ProcessExitMonitor {
private:
    struct ProcessInfo {
        int pid;
        std::string commandLine;
        std::chrono::system_clock::time_point startTime;
        std::chrono::system_clock::time_point endTime;
        int exitCode;
        std::string exitReason;
        
        ProcessInfo(int p, const std::string& cmd) 
            : pid(p), commandLine(cmd), exitCode(-1) {
            startTime = std::chrono::system_clock::now();
        }
    };
    
    std::map<int, std::unique_ptr<ProcessInfo>> processes;
    std::ofstream logFile;
    
public:
    ProcessExitMonitor(const std::string& logFileName = "process_monitor.log") {
        logFile.open(logFileName, std::ios::app);
        if (logFile.is_open()) {
            auto now = std::chrono::system_clock::now();
            auto time_t = std::chrono::system_clock::to_time_t(now);
            logFile << "\n=== Process Monitor Started at " << std::ctime(&time_t) << "===" << std::endl;
        }
    }
    
    ~ProcessExitMonitor() {
        if (logFile.is_open()) {
            auto now = std::chrono::system_clock::now();
            auto time_t = std::chrono::system_clock::to_time_t(now);
            logFile << "=== Process Monitor Stopped at " << std::ctime(&time_t) << "===" << std::endl;
        }
    }
    
    void registerProcess(int pid, const std::string& commandLine) {
        processes[pid] = std::make_unique<ProcessInfo>(pid, commandLine);
        logProcess(pid, "STARTED", "Process registered");
    }
    
    void recordProcessExit(int pid, int exitCode, const std::string& reason = "") {
        auto it = processes.find(pid);
        if (it != processes.end()) {
            it->second->endTime = std::chrono::system_clock::now();
            it->second->exitCode = exitCode;
            it->second->exitReason = reason.empty() ? getExitCodeDescription(exitCode) : reason;
            
            logProcess(pid, "EXITED", "Exit code: " + std::to_string(exitCode) + 
                      (reason.empty() ? "" : ", Reason: " + reason));
        }
    }
    
    void generateReport() const {
        std::cout << "\n=== Process Execution Report ===" << std::endl;
        
        for (const auto& pair : processes) {
            const ProcessInfo& info = *pair.second;
            
            auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(
                info.endTime - info.startTime).count();
            
            std::cout << "\nProcess ID: " << info.pid << std::endl;
            std::cout << "Command: " << info.commandLine << std::endl;
            std::cout << "Duration: " << duration << " ms" << std::endl;
            std::cout << "Exit Code: " << info.exitCode << std::endl;
            std::cout << "Exit Reason: " << info.exitReason << std::endl;
            std::cout << "Status: " << (info.exitCode == 0 ? "SUCCESS" : "FAILED") << std::endl;
        }
    }
    
private:
    void logProcess(int pid, const std::string& event, const std::string& details) {
        if (logFile.is_open()) {
            auto now = std::chrono::system_clock::now();
            auto time_t = std::chrono::system_clock::to_time_t(now);
            auto tm = *std::localtime(&time_t);
            
            logFile << "[" << std::put_time(&tm, "%Y-%m-%d %H:%M:%S") << "] "
                    << "PID " << pid << " " << event << ": " << details << std::endl;
        }
    }
    
    std::string getExitCodeDescription(int exitCode) const {
        switch (exitCode) {
            case 0: return "Success";
            case 1: return "General error";
            case 2: return "Misuse of shell command";
            case 126: return "Command invoked cannot execute";
            case 127: return "Command not found";
            case 128: return "Invalid argument to exit";
            case 130: return "Script terminated by Control-C";
            default: 
                if (exitCode > 128) {
                    return "Terminated by signal " + std::to_string(exitCode - 128);
                }
                return "Unknown error";
        }
    }
};

// Cross-platform process executor
class ProcessExecutor {
private:
    ProcessExitMonitor& monitor;
    
public:
    ProcessExecutor(ProcessExitMonitor& mon) : monitor(mon) {}
    
    struct ExecutionResult {
        int exitCode;
        std::string output;
        std::string error;
        std::chrono::milliseconds duration;
        bool success;
        
        ExecutionResult() : exitCode(-1), duration(0), success(false) {}
    };
    
    ExecutionResult executeCommand(const std::string& command, 
                                  int timeoutSeconds = 30) {
        ExecutionResult result;
        auto startTime = std::chrono::steady_clock::now();
        
        std::cout << "Executing command: " << command << std::endl;
        
        // Simulate process execution (in real implementation, use system() or CreateProcess)
        result.exitCode = simulateCommandExecution(command);
        
        auto endTime = std::chrono::steady_clock::now();
        result.duration = std::chrono::duration_cast<std::chrono::milliseconds>(endTime - startTime);
        result.success = (result.exitCode == 0);
        
        // Register with monitor
        int simulatedPid = rand() % 10000 + 1000;  // Simulate PID
        monitor.registerProcess(simulatedPid, command);
        monitor.recordProcessExit(simulatedPid, result.exitCode);
        
        return result;
    }
    
    std::vector<ExecutionResult> executeBatch(const std::vector<std::string>& commands,
                                            bool stopOnError = true) {
        std::vector<ExecutionResult> results;
        
        for (const auto& command : commands) {
            ExecutionResult result = executeCommand(command);
            results.push_back(result);
            
            if (stopOnError && !result.success) {
                std::cerr << "Batch execution stopped due to error in command: " << command << std::endl;
                break;
            }
        }
        
        return results;
    }
    
private:
    int simulateCommandExecution(const std::string& command) {
        // Simulate different command execution scenarios
        std::this_thread::sleep_for(std::chrono::milliseconds(100 + rand() % 500));
        
        if (command.find("success") != std::string::npos) {
            return 0;
        } else if (command.find("not_found") != std::string::npos) {
            return 127;  // Command not found
        } else if (command.find("permission") != std::string::npos) {
            return 126;  // Permission denied
        } else if (command.find("timeout") != std::string::npos) {
            return 124;  // Timeout
        } else if (command.find("invalid") != std::string::npos) {
            return 2;    // Invalid usage
        } else if (command.find("crash") != std::string::npos) {
            return 139;  // Segmentation fault (128 + 11)
        } else {
            // Random success/failure for demonstration
            return (rand() % 4 == 0) ? 1 : 0;
        }
    }
};

// Application configuration and validation
class ApplicationConfig {
private:
    std::map<std::string, std::string> config;
    std::vector<std::string> errors;
    
public:
    bool loadConfig(const std::string& configFile) {
        std::cout << "Loading configuration from: " << configFile << std::endl;
        
        // Simulate config loading
        config["database_host"] = "localhost";
        config["database_port"] = "5432";
        config["log_level"] = "INFO";
        config["max_connections"] = "100";
        
        // Simulate configuration errors
        if (configFile.find("invalid") != std::string::npos) {
            errors.push_back("Invalid database host");
            errors.push_back("Missing required API key");
            return false;
        }
        
        if (configFile.find("missing") != std::string::npos) {
            errors.push_back("Configuration file not found");
            return false;
        }
        
        return validateConfig();
    }
    
    bool validateConfig() {
        errors.clear();
        
        // Validate database port
        try {
            int port = std::stoi(config["database_port"]);
            if (port <= 0 || port > 65535) {
                errors.push_back("Invalid database port: " + config["database_port"]);
            }
        } catch (const std::exception&) {
            errors.push_back("Database port must be a number");
        }
        
        // Validate max connections
        try {
            int maxConn = std::stoi(config["max_connections"]);
            if (maxConn <= 0 || maxConn > 1000) {
                errors.push_back("Invalid max_connections: " + config["max_connections"]);
            }
        } catch (const std::exception&) {
            errors.push_back("max_connections must be a number");
        }
        
        // Validate log level
        std::vector<std::string> validLevels = {"DEBUG", "INFO", "WARN", "ERROR"};
        if (std::find(validLevels.begin(), validLevels.end(), config["log_level"]) == validLevels.end()) {
            errors.push_back("Invalid log_level: " + config["log_level"]);
        }
        
        return errors.empty();
    }
    
    void printErrors() const {
        if (!errors.empty()) {
            std::cerr << "Configuration errors:" << std::endl;
            for (const auto& error : errors) {
                std::cerr << "  - " << error << std::endl;
            }
        }
    }
    
    std::string getValue(const std::string& key) const {
        auto it = config.find(key);
        return (it != config.end()) ? it->second : "";
    }
    
    bool hasErrors() const { return !errors.empty(); }
};

// Main application controller with comprehensive exit handling
class ApplicationController {
private:
    ProcessExitMonitor monitor;
    ProcessExecutor executor;
    ApplicationConfig config;
    
public:
    ApplicationController() : executor(monitor) {}
    
    int run(int argc, char* argv[]) {
        try {
            // Parse command line arguments
            if (argc < 2) {
                printUsage(argv[0]);
                return ExitCodes::INVALID_ARGUMENTS;
            }
            
            std::string command = argv[1];
            
            if (command == "--help" || command == "-h") {
                printUsage(argv[0]);
                return ExitCodes::SUCCESS;
            }
            
            if (command == "config") {
                return handleConfigCommand(argc, argv);
            } else if (command == "execute") {
                return handleExecuteCommand(argc, argv);
            } else if (command == "batch") {
                return handleBatchCommand(argc, argv);
            } else if (command == "test") {
                return handleTestCommand(argc, argv);
            } else {
                std::cerr << "Unknown command: " << command << std::endl;
                printUsage(argv[0]);
                return ExitCodes::INVALID_ARGUMENTS;
            }
            
        } catch (const std::exception& e) {
            std::cerr << "Unexpected error: " << e.what() << std::endl;
            return ExitCodes::GENERAL_ERROR;
        }
    }
    
private:
    void printUsage(const char* programName) const {
        std::cout << "Usage: " << programName << " <command> [options]" << std::endl;
        std::cout << "\nCommands:" << std::endl;
        std::cout << "  config <file>        Load and validate configuration" << std::endl;
        std::cout << "  execute <command>    Execute a single command" << std::endl;
        std::cout << "  batch <file>         Execute commands from file" << std::endl;
        std::cout << "  test                 Run system tests" << std::endl;
        std::cout << "  --help, -h          Show this help message" << std::endl;
        std::cout << "\nExit Codes:" << std::endl;
        ExitCodeManager::listAllExitCodes();
    }
    
    int handleConfigCommand(int argc, char* argv[]) {
        if (argc < 3) {
            std::cerr << "Config file path required" << std::endl;
            return ExitCodes::INVALID_ARGUMENTS;
        }
        
        std::string configFile = argv[2];
        
        if (!config.loadConfig(configFile)) {
            config.printErrors();
            return ExitCodes::CONFIGURATION_ERROR;
        }
        
        std::cout << "Configuration loaded and validated successfully" << std::endl;
        return ExitCodes::SUCCESS;
    }
    
    int handleExecuteCommand(int argc, char* argv[]) {
        if (argc < 3) {
            std::cerr << "Command to execute required" << std::endl;
            return ExitCodes::INVALID_ARGUMENTS;
        }
        
        std::string command = argv[2];
        auto result = executor.executeCommand(command);
        
        std::cout << "Command execution completed" << std::endl;
        std::cout << "Exit code: " << result.exitCode << std::endl;
        std::cout << "Duration: " << result.duration.count() << " ms" << std::endl;
        std::cout << "Success: " << (result.success ? "Yes" : "No") << std::endl;
        
        return result.success ? ExitCodes::SUCCESS : ExitCodes::GENERAL_ERROR;
    }
    
    int handleBatchCommand(int argc, char* argv[]) {
        if (argc < 3) {
            std::cerr << "Batch file path required" << std::endl;
            return ExitCodes::INVALID_ARGUMENTS;
        }
        
        std::string batchFile = argv[2];
        std::vector<std::string> commands = loadBatchFile(batchFile);
        
        if (commands.empty()) {
            std::cerr << "No commands found in batch file or file not accessible" << std::endl;
            return ExitCodes::FILE_NOT_FOUND;
        }
        
        auto results = executor.executeBatch(commands, true);
        
        // Analyze results
        int successCount = 0;
        int failureCount = 0;
        
        for (const auto& result : results) {
            if (result.success) {
                successCount++;
            } else {
                failureCount++;
            }
        }
        
        std::cout << "\nBatch execution summary:" << std::endl;
        std::cout << "Total commands: " << results.size() << std::endl;
        std::cout << "Successful: " << successCount << std::endl;
        std::cout << "Failed: " << failureCount << std::endl;
        
        monitor.generateReport();
        
        return (failureCount > 0) ? ExitCodes::GENERAL_ERROR : ExitCodes::SUCCESS;
    }
    
    int handleTestCommand(int argc, char* argv[]) {
        std::cout << "Running system tests..." << std::endl;
        
        std::vector<std::string> testCommands = {
            "test_success",
            "test_invalid_args",
            "test_not_found",
            "test_permission_denied",
            "test_timeout"
        };
        
        auto results = executor.executeBatch(testCommands, false);  // Don't stop on error
        
        int passedTests = 0;
        int failedTests = 0;
        
        for (size_t i = 0; i < results.size(); i++) {
            const auto& result = results[i];
            std::cout << "Test " << (i + 1) << " (" << testCommands[i] << "): ";
            
            if (result.success) {
                std::cout << "PASSED" << std::endl;
                passedTests++;
            } else {
                std::cout << "FAILED (exit code: " << result.exitCode << ")" << std::endl;
                failedTests++;
            }
        }
        
        std::cout << "\nTest Results:" << std::endl;
        std::cout << "Passed: " << passedTests << std::endl;
        std::cout << "Failed: " << failedTests << std::endl;
        std::cout << "Total: " << results.size() << std::endl;
        
        return (failedTests > 0) ? ExitCodes::GENERAL_ERROR : ExitCodes::SUCCESS;
    }
    
    std::vector<std::string> loadBatchFile(const std::string& filename) {
        std::vector<std::string> commands;
        
        // Simulate batch file content
        if (filename.find("missing") != std::string::npos) {
            return commands;  // Empty - file not found
        }
        
        // Simulate loading commands from file
        commands.push_back("echo Starting batch execution");
        commands.push_back("test_success");
        commands.push_back("test_another_success");
        commands.push_back("test_failure");
        commands.push_back("echo Batch execution completed");
        
        return commands;
    }
};

void demonstrateSystemIntegration() {
    std::cout << "\n=== System Integration Demo ===" << std::endl;
    
    ProcessExitMonitor monitor("demo_monitor.log");
    ProcessExecutor executor(monitor);
    
    // Test different command scenarios
    std::vector<std::string> testCommands = {
        "echo success",
        "command_not_found",
        "permission_denied_command",
        "timeout_command",
        "crash_command"
    };
    
    std::cout << "\nExecuting test commands..." << std::endl;
    auto results = executor.executeBatch(testCommands, false);
    
    // Generate comprehensive report
    monitor.generateReport();
    
    // Summary statistics
    int successful = 0;
    for (const auto& result : results) {
        if (result.success) successful++;
    }
    
    std::cout << "\nExecution Summary:" << std::endl;
    std::cout << "Total commands: " << results.size() << std::endl;
    std::cout << "Successful: " << successful << std::endl;
    std::cout << "Failed: " << (results.size() - successful) << std::endl;
}

int main(int argc, char* argv[]) {
    // Initialize random seed for simulation
    srand(static_cast<unsigned>(time(nullptr)));
    
    ApplicationController app;
    
    // If no arguments provided, run demonstration
    if (argc == 1) {
        demonstrateSystemIntegration();
        return ExitCodes::SUCCESS;
    }
    
    // Otherwise, run the application normally
    return app.run(argc, argv);
}
```

## Error Reporting ve Logging

### Comprehensive Error Tracking System

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <map>
#include <memory>
#include <chrono>
#include <iomanip>
#include <sstream>
#include <thread>
#include <mutex>

// Advanced error reporting and logging system
enum class LogLevel {
    TRACE = 0,
    DEBUG = 1,
    INFO = 2,
    WARN = 3,
    ERROR = 4,
    FATAL = 5
};

enum class ErrorCategory {
    SYSTEM,
    NETWORK,
    DATABASE,
    VALIDATION,
    SECURITY,
    CONFIGURATION,
    USER_INPUT,
    BUSINESS_LOGIC
};

struct ErrorInfo {
    int errorCode;
    ErrorCategory category;
    LogLevel severity;
    std::string message;
    std::string location;
    std::chrono::system_clock::time_point timestamp;
    std::map<std::string, std::string> context;
    
    ErrorInfo(int code, ErrorCategory cat, LogLevel sev, const std::string& msg, const std::string& loc)
        : errorCode(code), category(cat), severity(sev), message(msg), location(loc) {
        timestamp = std::chrono::system_clock::now();
    }
    
    void addContext(const std::string& key, const std::string& value) {
        context[key] = value;
    }
    
    std::string toString() const {
        std::ostringstream oss;
        auto time_t = std::chrono::system_clock::to_time_t(timestamp);
        auto tm = *std::localtime(&time_t);
        
        oss << "[" << std::put_time(&tm, "%Y-%m-%d %H:%M:%S") << "] ";
        oss << logLevelToString(severity) << " ";
        oss << "(" << categoryToString(category) << ") ";
        oss << "Code " << errorCode << ": " << message;
        oss << " [" << location << "]";
        
        if (!context.empty()) {
            oss << " Context: {";
            bool first = true;
            for (const auto& pair : context) {
                if (!first) oss << ", ";
                oss << pair.first << "=" << pair.second;
                first = false;
            }
            oss << "}";
        }
        
        return oss.str();
    }
    
private:
    std::string logLevelToString(LogLevel level) const {
        switch (level) {
            case LogLevel::TRACE: return "TRACE";
            case LogLevel::DEBUG: return "DEBUG";
            case LogLevel::INFO:  return "INFO ";
            case LogLevel::WARN:  return "WARN ";
            case LogLevel::ERROR: return "ERROR";
            case LogLevel::FATAL: return "FATAL";
            default: return "UNKNOWN";
        }
    }
    
    std::string categoryToString(ErrorCategory cat) const {
        switch (cat) {
            case ErrorCategory::SYSTEM: return "SYSTEM";
            case ErrorCategory::NETWORK: return "NETWORK";
            case ErrorCategory::DATABASE: return "DATABASE";
            case ErrorCategory::VALIDATION: return "VALIDATION";
            case ErrorCategory::SECURITY: return "SECURITY";
            case ErrorCategory::CONFIGURATION: return "CONFIG";
            case ErrorCategory::USER_INPUT: return "USER_INPUT";
            case ErrorCategory::BUSINESS_LOGIC: return "BUSINESS";
            default: return "UNKNOWN";
        }
    }
};

class ErrorLogger {
private:
    std::ofstream logFile;
    std::mutex logMutex;
    LogLevel minLogLevel;
    std::vector<std::unique_ptr<ErrorInfo>> errorHistory;
    std::map<ErrorCategory, int> errorCounts;
    bool consoleOutput;
    
public:
    ErrorLogger(const std::string& logFileName = "error.log", 
                LogLevel minLevel = LogLevel::INFO,
                bool enableConsole = true)
        : minLogLevel(minLevel), consoleOutput(enableConsole) {
        
        logFile.open(logFileName, std::ios::app);
        if (logFile.is_open()) {
            auto now = std::chrono::system_clock::now();
            auto time_t = std::chrono::system_clock::to_time_t(now);
            logFile << "\n=== Error Logger Started at " << std::ctime(&time_t) << "===" << std::endl;
        }
    }
    
    ~ErrorLogger() {
        if (logFile.is_open()) {
            auto now = std::chrono::system_clock::now();
            auto time_t = std::chrono::system_clock::to_time_t(now);
            logFile << "=== Error Logger Stopped at " << std::ctime(&time_t) << "===" << std::endl;
        }
    }
    
    void logError(const ErrorInfo& error) {
        if (error.severity < minLogLevel) {
            return;
        }
        
        std::lock_guard<std::mutex> lock(logMutex);
        
        // Store error in history
        auto errorCopy = std::make_unique<ErrorInfo>(error);
        errorHistory.push_back(std::move(errorCopy));
        
        // Update category counts
        errorCounts[error.category]++;
        
        // Write to log file
        if (logFile.is_open()) {
            logFile << error.toString() << std::endl;
            logFile.flush();
        }
        
        // Write to console if enabled
        if (consoleOutput) {
            if (error.severity >= LogLevel::ERROR) {
                std::cerr << error.toString() << std::endl;
            } else {
                std::cout << error.toString() << std::endl;
            }
        }
    }
    
    void logError(int code, ErrorCategory category, LogLevel severity,
                  const std::string& message, const std::string& location = "",
                  const std::map<std::string, std::string>& context = {}) {
        ErrorInfo error(code, category, severity, message, location);
        for (const auto& pair : context) {
            error.addContext(pair.first, pair.second);
        }
        logError(error);
    }
    
    void generateReport() const {
        std::cout << "\n=== Error Report ===" << std::endl;
        std::cout << "Total errors logged: " << errorHistory.size() << std::endl;
        
        std::cout << "\nErrors by category:" << std::endl;
        for (const auto& pair : errorCounts) {
            std::cout << "  " << categoryToString(pair.first) << ": " << pair.second << std::endl;
        }
        
        std::cout << "\nErrors by severity:" << std::endl;
        std::map<LogLevel, int> severityCounts;
        for (const auto& error : errorHistory) {
            severityCounts[error->severity]++;
        }
        
        for (const auto& pair : severityCounts) {
            std::cout << "  " << logLevelToString(pair.first) << ": " << pair.second << std::endl;
        }
        
        // Recent errors
        std::cout << "\nRecent errors (last 5):" << std::endl;
        size_t start = (errorHistory.size() > 5) ? errorHistory.size() - 5 : 0;
        for (size_t i = start; i < errorHistory.size(); i++) {
            std::cout << "  " << errorHistory[i]->toString() << std::endl;
        }
    }
    
    std::vector<const ErrorInfo*> getErrorsByCategory(ErrorCategory category) const {
        std::vector<const ErrorInfo*> result;
        for (const auto& error : errorHistory) {
            if (error->category == category) {
                result.push_back(error.get());
            }
        }
        return result;
    }
    
    std::vector<const ErrorInfo*> getErrorsBySeverity(LogLevel severity) const {
        std::vector<const ErrorInfo*> result;
        for (const auto& error : errorHistory) {
            if (error->severity >= severity) {
                result.push_back(error.get());
            }
        }
        return result;
    }
    
    bool hasErrorsOfSeverity(LogLevel severity) const {
        for (const auto& error : errorHistory) {
            if (error->severity >= severity) {
                return true;
            }
        }
        return false;
    }
    
    int getExitCodeBasedOnErrors() const {
        if (hasErrorsOfSeverity(LogLevel::FATAL)) {
            return ExitCodes::GENERAL_ERROR;
        }
        if (hasErrorsOfSeverity(LogLevel::ERROR)) {
            return ExitCodes::GENERAL_ERROR;
        }
        if (hasErrorsOfSeverity(LogLevel::WARN)) {
            return ExitCodes::SUCCESS;  // Warnings don't cause failure
        }
        return ExitCodes::SUCCESS;
    }
    
private:
    std::string categoryToString(ErrorCategory cat) const {
        switch (cat) {
            case ErrorCategory::SYSTEM: return "SYSTEM";
            case ErrorCategory::NETWORK: return "NETWORK";
            case ErrorCategory::DATABASE: return "DATABASE";
            case ErrorCategory::VALIDATION: return "VALIDATION";
            case ErrorCategory::SECURITY: return "SECURITY";
            case ErrorCategory::CONFIGURATION: return "CONFIG";
            case ErrorCategory::USER_INPUT: return "USER_INPUT";
            case ErrorCategory::BUSINESS_LOGIC: return "BUSINESS";
            default: return "UNKNOWN";
        }
    }
    
    std::string logLevelToString(LogLevel level) const {
        switch (level) {
            case LogLevel::TRACE: return "TRACE";
            case LogLevel::DEBUG: return "DEBUG";
            case LogLevel::INFO:  return "INFO ";
            case LogLevel::WARN:  return "WARN ";
            case LogLevel::ERROR: return "ERROR";
            case LogLevel::FATAL: return "FATAL";
            default: return "UNKNOWN";
        }
    }
};

// Application with comprehensive error handling
class RobustApplication {
private:
    ErrorLogger logger;
    std::string appName;
    
public:
    RobustApplication(const std::string& name) 
        : logger(name + "_errors.log", LogLevel::DEBUG, true), appName(name) {
        logger.logError(1000, ErrorCategory::SYSTEM, LogLevel::INFO, 
                       "Application started", "RobustApplication::constructor",
                       {{"app_name", appName}, {"version", "1.0.0"}});
    }
    
    ~RobustApplication() {
        logger.logError(1001, ErrorCategory::SYSTEM, LogLevel::INFO, 
                       "Application shutting down", "RobustApplication::destructor");
        logger.generateReport();
    }
    
    int processFile(const std::string& filename) {
        logger.logError(2000, ErrorCategory::SYSTEM, LogLevel::INFO, 
                       "Starting file processing", "processFile",
                       {{"filename", filename}});
        
        // Validate input
        if (filename.empty()) {
            logger.logError(2001, ErrorCategory::VALIDATION, LogLevel::ERROR,
                           "Filename cannot be empty", "processFile::validation");
            return ExitCodes::INVALID_ARGUMENTS;
        }
        
        // Simulate file operations with various error scenarios
        if (filename.find("not_found") != std::string::npos) {
            logger.logError(2002, ErrorCategory::SYSTEM, LogLevel::ERROR,
                           "File not found", "processFile::file_access",
                           {{"filename", filename}, {"operation", "open"}});
            return ExitCodes::FILE_NOT_FOUND;
        }
        
        if (filename.find("permission") != std::string::npos) {
            logger.logError(2003, ErrorCategory::SECURITY, LogLevel::ERROR,
                           "Permission denied", "processFile::file_access",
                           {{"filename", filename}, {"user", "current_user"}});
            return ExitCodes::PERMISSION_DENIED;
        }
        
        if (filename.find("corrupt") != std::string::npos) {
            logger.logError(2004, ErrorCategory::VALIDATION, LogLevel::ERROR,
                           "File content is corrupted", "processFile::validation",
                           {{"filename", filename}, {"error_type", "corruption"}});
            return ExitCodes::VALIDATION_ERROR;
        }
        
        // Simulate successful processing with some warnings
        logger.logError(2005, ErrorCategory::SYSTEM, LogLevel::WARN,
                       "File format is deprecated", "processFile::format_check",
                       {{"filename", filename}, {"format", "legacy"}});
        
        logger.logError(2006, ErrorCategory::SYSTEM, LogLevel::INFO,
                       "File processed successfully", "processFile::completion",
                       {{"filename", filename}, {"lines_processed", "1000"}});
        
        return ExitCodes::SUCCESS;
    }
    
    int connectToDatabase(const std::string& connectionString) {
        logger.logError(3000, ErrorCategory::DATABASE, LogLevel::INFO,
                       "Attempting database connection", "connectToDatabase",
                       {{"connection_string", connectionString}});
        
        // Validate connection string
        if (connectionString.empty()) {
            logger.logError(3001, ErrorCategory::VALIDATION, LogLevel::ERROR,
                           "Connection string is empty", "connectToDatabase::validation");
            return ExitCodes::CONFIGURATION_ERROR;
        }
        
        // Simulate different database scenarios
        if (connectionString.find("invalid_host") != std::string::npos) {
            logger.logError(3002, ErrorCategory::NETWORK, LogLevel::ERROR,
                           "Cannot resolve database host", "connectToDatabase::dns",
                           {{"host", "invalid_host"}, {"timeout", "5s"}});
            return ExitCodes::NETWORK_ERROR;
        }
        
        if (connectionString.find("auth_failed") != std::string::npos) {
            logger.logError(3003, ErrorCategory::SECURITY, LogLevel::ERROR,
                           "Authentication failed", "connectToDatabase::auth",
                           {{"reason", "invalid_credentials"}});
            return ExitCodes::PERMISSION_DENIED;
        }
        
        if (connectionString.find("timeout") != std::string::npos) {
            logger.logError(3004, ErrorCategory::NETWORK, LogLevel::ERROR,
                           "Connection timeout", "connectToDatabase::timeout",
                           {{"timeout", "30s"}});
            return ExitCodes::TIMEOUT_ERROR;
        }
        
        // Successful connection
        logger.logError(3005, ErrorCategory::DATABASE, LogLevel::INFO,
                       "Database connection established", "connectToDatabase::success",
                       {{"database", "myapp_db"}, {"version", "13.2"}});
        
        return ExitCodes::SUCCESS;
    }
    
    int validateUserInput(const std::string& input) {
        logger.logError(4000, ErrorCategory::USER_INPUT, LogLevel::DEBUG,
                       "Validating user input", "validateUserInput",
                       {{"input_length", std::to_string(input.length())}});
        
        std::vector<std::string> validationErrors;
        
        // Length validation
        if (input.length() < 3) {
            validationErrors.push_back("Input too short (minimum 3 characters)");
        }
        
        if (input.length() > 100) {
            validationErrors.push_back("Input too long (maximum 100 characters)");
        }
        
        // Content validation
        if (input.find_first_not_of("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_-") != std::string::npos) {
            validationErrors.push_back("Input contains invalid characters");
        }
        
        // SQL injection check (simple)
        std::vector<std::string> sqlKeywords = {"DROP", "DELETE", "INSERT", "UPDATE", "SELECT"};
        for (const auto& keyword : sqlKeywords) {
            if (input.find(keyword) != std::string::npos) {
                validationErrors.push_back("Input contains potential SQL injection");
                logger.logError(4001, ErrorCategory::SECURITY, LogLevel::WARN,
                               "Potential SQL injection detected", "validateUserInput::security",
                               {{"input", input}, {"keyword", keyword}});
            }
        }
        
        // Log validation results
        if (!validationErrors.empty()) {
            for (const auto& error : validationErrors) {
                logger.logError(4002, ErrorCategory::VALIDATION, LogLevel::ERROR,
                               error, "validateUserInput::validation",
                               {{"input", input}});
            }
            return ExitCodes::VALIDATION_ERROR;
        }
        
        logger.logError(4003, ErrorCategory::USER_INPUT, LogLevel::INFO,
                       "User input validation passed", "validateUserInput::success",
                       {{"input", input}});
        
        return ExitCodes::SUCCESS;
    }
    
    void demonstrateComprehensiveErrorHandling() {
        std::cout << "\n=== Comprehensive Error Handling Demo ===" << std::endl;
        
        // Test various scenarios
        std::vector<std::pair<std::string, std::string>> fileTests = {
            {"valid_file.txt", "Valid file"},
            {"not_found_file.txt", "File not found"},
            {"permission_denied.txt", "Permission denied"},
            {"corrupt_file.dat", "Corrupted file"}
        };
        
        for (const auto& test : fileTests) {
            std::cout << "\nTesting: " << test.second << std::endl;
            int result = processFile(test.first);
            std::cout << "Result: " << result << " (" << 
                        (result == ExitCodes::SUCCESS ? "SUCCESS" : "FAILED") << ")" << std::endl;
        }
        
        // Test database connections
        std::vector<std::pair<std::string, std::string>> dbTests = {
            {"host=localhost;db=myapp", "Valid connection"},
            {"host=invalid_host;db=test", "Invalid host"},
            {"host=localhost;user=wrong;db=test", "Auth failed"},
            {"host=timeout_server;db=test", "Connection timeout"}
        };
        
        for (const auto& test : dbTests) {
            std::cout << "\nTesting: " << test.second << std::endl;
            int result = connectToDatabase(test.first);
            std::cout << "Result: " << result << " (" << 
                        (result == ExitCodes::SUCCESS ? "SUCCESS" : "FAILED") << ")" << std::endl;
        }
        
        // Test input validation
        std::vector<std::pair<std::string, std::string>> inputTests = {
            {"valid_input123", "Valid input"},
            {"ab", "Too short"},
            {"input_with_special_chars!", "Invalid characters"},
            {"DROP TABLE users", "SQL injection attempt"}
        };
        
        for (const auto& test : inputTests) {
            std::cout << "\nTesting: " << test.second << std::endl;
            int result = validateUserInput(test.first);
            std::cout << "Result: " << result << " (" << 
                        (result == ExitCodes::SUCCESS ? "SUCCESS" : "FAILED") << ")" << std::endl;
        }
    }
    
    int getOverallExitCode() const {
        return logger.getExitCodeBasedOnErrors();
    }
};

void demonstrateErrorReportingSystem() {
    std::cout << "\n=== Error Reporting and Logging Demo ===" << std::endl;
    
    RobustApplication app("DemoApp");
    
    // Run comprehensive error handling demonstration
    app.demonstrateComprehensiveErrorHandling();
    
    // Final exit code based on accumulated errors
    int finalExitCode = app.getOverallExitCode();
    std::cout << "\nFinal application exit code: " << finalExitCode << std::endl;
    
    ExitCodeManager::printExitCodeInfo(finalExitCode);
}

int main(int argc, char* argv[]) {
    // System integration demo
    demonstrateSystemIntegration();
    
    // Error reporting demo
    demonstrateErrorReportingSystem();
    
    std::cout << "\n=== Exit Code Best Practices ===" << std::endl;
    std::cout << "✓ Use standard exit codes when possible (0 = success, 1 = error)" << std::endl;
    std::cout << "✓ Define application-specific exit codes for different error types" << std::endl;
    std::cout << "✓ Document all exit codes in your application" << std::endl;
    std::cout << "✓ Use exit codes consistently across your application" << std::endl;
    std::cout << "✓ Log detailed error information along with exit codes" << std::endl;
    std::cout << "✓ Consider the calling environment (scripts, CI/CD) when designing exit codes" << std::endl;
    std::cout << "✓ Test exit code behavior in different scenarios" << std::endl;
    
    return ExitCodes::SUCCESS;
}
```

Çıkış kodları, modern yazılım geliştirmede system integration ve automation için kritik öneme sahiptir. Doğru kullanıldığında debugging, monitoring ve error handling süreçlerini büyük ölçüde kolaylaştırır.
