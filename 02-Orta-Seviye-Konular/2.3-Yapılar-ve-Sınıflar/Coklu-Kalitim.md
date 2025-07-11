# Çoklu Kalıtım (Multiple Inheritance)

Çoklu kalıtım, C++'da bir sınıfın birden fazla temel sınıftan kalıtım alabilmesi özelliğidir. Bu güçlü ancak karmaşık bir yapı olup, doğru kullanıldığında esneklik sağlar, yanlış kullanıldığında ise sorunlara yol açabilir.

## Temel Çoklu Kalıtım

### Basit Çoklu Kalıtım

```cpp
#include <iostream>
#include <string>
#include <memory>

// İlk temel sınıf - Flyable
class Flyable {
protected:
    double altitude;
    double maxAltitude;
    
public:
    Flyable(double maxAlt = 10000.0) : altitude(0.0), maxAltitude(maxAlt) {
        std::cout << "Flyable constructor called" << std::endl;
    }
    
    virtual ~Flyable() {
        std::cout << "Flyable destructor called" << std::endl;
    }
    
    virtual void takeOff() {
        altitude = 100.0;
        std::cout << "Taking off to altitude: " << altitude << std::endl;
    }
    
    virtual void land() {
        altitude = 0.0;
        std::cout << "Landing - altitude: " << altitude << std::endl;
    }
    
    virtual void fly(double newAltitude) {
        if (newAltitude <= maxAltitude) {
            altitude = newAltitude;
            std::cout << "Flying at altitude: " << altitude << std::endl;
        } else {
            std::cout << "Cannot fly above max altitude: " << maxAltitude << std::endl;
        }
    }
    
    double getAltitude() const { return altitude; }
    double getMaxAltitude() const { return maxAltitude; }
    
    virtual void showFlightInfo() const {
        std::cout << "Flight Info - Current: " << altitude 
                  << ", Max: " << maxAltitude << std::endl;
    }
};

// İkinci temel sınıf - Swimmable
class Swimmable {
protected:
    double depth;
    double maxDepth;
    bool underwater;
    
public:
    Swimmable(double maxDep = 500.0) : depth(0.0), maxDepth(maxDep), underwater(false) {
        std::cout << "Swimmable constructor called" << std::endl;
    }
    
    virtual ~Swimmable() {
        std::cout << "Swimmable destructor called" << std::endl;
    }
    
    virtual void dive() {
        underwater = true;
        depth = 10.0;
        std::cout << "Diving to depth: " << depth << std::endl;
    }
    
    virtual void surface() {
        depth = 0.0;
        underwater = false;
        std::cout << "Surfacing - depth: " << depth << std::endl;
    }
    
    virtual void swim(double newDepth) {
        if (newDepth <= maxDepth) {
            depth = newDepth;
            underwater = (depth > 0);
            std::cout << "Swimming at depth: " << depth << std::endl;
        } else {
            std::cout << "Cannot swim below max depth: " << maxDepth << std::endl;
        }
    }
    
    double getDepth() const { return depth; }
    double getMaxDepth() const { return maxDepth; }
    bool isUnderwater() const { return underwater; }
    
    virtual void showSwimInfo() const {
        std::cout << "Swim Info - Current: " << depth 
                  << ", Max: " << maxDepth 
                  << ", Underwater: " << (underwater ? "Yes" : "No") << std::endl;
    }
};

// Üçüncü temel sınıf - Vehicle
class Vehicle {
protected:
    std::string name;
    double speed;
    double maxSpeed;
    bool engineRunning;
    
public:
    Vehicle(const std::string& n, double maxSpd = 100.0) 
        : name(n), speed(0.0), maxSpeed(maxSpd), engineRunning(false) {
        std::cout << "Vehicle constructor called for: " << name << std::endl;
    }
    
    virtual ~Vehicle() {
        std::cout << "Vehicle destructor called for: " << name << std::endl;
    }
    
    virtual void startEngine() {
        engineRunning = true;
        std::cout << name << " engine started" << std::endl;
    }
    
    virtual void stopEngine() {
        engineRunning = false;
        speed = 0.0;
        std::cout << name << " engine stopped" << std::endl;
    }
    
    virtual void accelerate(double newSpeed) {
        if (engineRunning && newSpeed <= maxSpeed) {
            speed = newSpeed;
            std::cout << name << " accelerating to: " << speed << " km/h" << std::endl;
        } else if (!engineRunning) {
            std::cout << "Cannot accelerate - engine not running" << std::endl;
        } else {
            std::cout << "Cannot exceed max speed: " << maxSpeed << std::endl;
        }
    }
    
    const std::string& getName() const { return name; }
    double getSpeed() const { return speed; }
    double getMaxSpeed() const { return maxSpeed; }
    bool isEngineRunning() const { return engineRunning; }
    
    virtual void showVehicleInfo() const {
        std::cout << "Vehicle: " << name 
                  << ", Speed: " << speed 
                  << ", Max Speed: " << maxSpeed
                  << ", Engine: " << (engineRunning ? "Running" : "Stopped") << std::endl;
    }
};

// Çoklu kalıtım - Amphibious Aircraft
class AmphibiousAircraft : public Flyable, public Swimmable, public Vehicle {
private:
    enum class Mode { GROUND, FLIGHT, WATER };
    Mode currentMode;
    
public:
    AmphibiousAircraft(const std::string& name) 
        : Flyable(15000.0), Swimmable(50.0), Vehicle(name, 300.0), currentMode(Mode::GROUND) {
        std::cout << "AmphibiousAircraft constructor called for: " << name << std::endl;
    }
    
    ~AmphibiousAircraft() {
        std::cout << "AmphibiousAircraft destructor called for: " << getName() << std::endl;
    }
    
    // Override virtual functions from all base classes
    void takeOff() override {
        if (currentMode == Mode::WATER) {
            surface();  // Surface first if in water
        }
        startEngine();
        Flyable::takeOff();
        currentMode = Mode::FLIGHT;
        std::cout << getName() << " is now in flight mode" << std::endl;
    }
    
    void land() override {
        Flyable::land();
        currentMode = Mode::GROUND;
        std::cout << getName() << " has landed" << std::endl;
    }
    
    void dive() override {
        if (currentMode == Mode::FLIGHT) {
            land();  // Land first if flying
        }
        startEngine();
        Swimmable::dive();
        currentMode = Mode::WATER;
        std::cout << getName() << " is now in water mode" << std::endl;
    }
    
    void surface() override {
        Swimmable::surface();
        currentMode = Mode::GROUND;
        std::cout << getName() << " has surfaced" << std::endl;
    }
    
    // New methods specific to amphibious aircraft
    void switchToFlightMode() {
        if (currentMode == Mode::WATER) {
            surface();
        }
        takeOff();
    }
    
    void switchToWaterMode() {
        if (currentMode == Mode::FLIGHT) {
            land();
        }
        dive();
    }
    
    void emergencyLanding() {
        std::cout << "EMERGENCY! " << getName() << " performing emergency landing!" << std::endl;
        if (currentMode == Mode::FLIGHT) {
            land();
        } else if (currentMode == Mode::WATER) {
            surface();
        }
        stopEngine();
    }
    
    Mode getCurrentMode() const { return currentMode; }
    
    std::string getModeString() const {
        switch (currentMode) {
            case Mode::GROUND: return "Ground";
            case Mode::FLIGHT: return "Flight";
            case Mode::WATER: return "Water";
            default: return "Unknown";
        }
    }
    
    // Comprehensive status display
    void showFullStatus() const {
        std::cout << "\n=== " << getName() << " Status ===" << std::endl;
        std::cout << "Current Mode: " << getModeString() << std::endl;
        showVehicleInfo();
        showFlightInfo();
        showSwimInfo();
        std::cout << "============================\n" << std::endl;
    }
};

void basicMultipleInheritanceDemo() {
    std::cout << "=== Basic Multiple Inheritance Demo ===" << std::endl;
    
    AmphibiousAircraft seaplane("SeaHawk-1");
    seaplane.showFullStatus();
    
    // Flight operations
    std::cout << "\n--- Flight Operations ---" << std::endl;
    seaplane.switchToFlightMode();
    seaplane.fly(5000);
    seaplane.accelerate(250);
    seaplane.showFullStatus();
    
    // Water operations
    std::cout << "\n--- Water Operations ---" << std::endl;
    seaplane.switchToWaterMode();
    seaplane.swim(25);
    seaplane.accelerate(80);
    seaplane.showFullStatus();
    
    // Emergency scenario
    std::cout << "\n--- Emergency Scenario ---" << std::endl;
    seaplane.switchToFlightMode();
    seaplane.fly(8000);
    seaplane.emergencyLanding();
    seaplane.showFullStatus();
}

int main() {
    basicMultipleInheritanceDemo();
    return 0;
}
```

## Diamond Problem ve Virtual Inheritance

### Diamond Problem

```cpp
#include <iostream>
#include <string>

// Temel sınıf - Animal
class Animal {
protected:
    std::string name;
    int age;
    double weight;
    
public:
    Animal(const std::string& n, int a, double w) : name(n), age(a), weight(w) {
        std::cout << "Animal constructor: " << name << std::endl;
    }
    
    virtual ~Animal() {
        std::cout << "Animal destructor: " << name << std::endl;
    }
    
    virtual void eat() {
        std::cout << name << " is eating" << std::endl;
    }
    
    virtual void sleep() {
        std::cout << name << " is sleeping" << std::endl;
    }
    
    virtual void makeSound() = 0;  // Pure virtual
    
    // Getters
    const std::string& getName() const { return name; }
    int getAge() const { return age; }
    double getWeight() const { return weight; }
    
    virtual void showInfo() const {
        std::cout << "Animal: " << name << ", Age: " << age << ", Weight: " << weight << "kg" << std::endl;
    }
};

// İlk türetilmiş sınıf - Mammal
class Mammal : public Animal {  // Normal inheritance - Diamond problem yaratır
protected:
    double bodyTemperature;
    
public:
    Mammal(const std::string& n, int a, double w, double temp = 37.0) 
        : Animal(n, a, w), bodyTemperature(temp) {
        std::cout << "Mammal constructor: " << name << std::endl;
    }
    
    virtual ~Mammal() {
        std::cout << "Mammal destructor: " << name << std::endl;
    }
    
    virtual void giveBirth() {
        std::cout << name << " is giving birth to live offspring" << std::endl;
    }
    
    virtual void regulateTemperature() {
        std::cout << name << " is regulating body temperature: " << bodyTemperature << "°C" << std::endl;
    }
    
    double getBodyTemperature() const { return bodyTemperature; }
    
    void showInfo() const override {
        Animal::showInfo();
        std::cout << "Body Temperature: " << bodyTemperature << "°C" << std::endl;
    }
};

// İkinci türetilmiş sınıf - Bird
class Bird : public Animal {  // Normal inheritance - Diamond problem yaratır
protected:
    double wingSpan;
    bool canFly;
    
public:
    Bird(const std::string& n, int a, double w, double ws, bool fly = true) 
        : Animal(n, a, w), wingSpan(ws), canFly(fly) {
        std::cout << "Bird constructor: " << name << std::endl;
    }
    
    virtual ~Bird() {
        std::cout << "Bird destructor: " << name << std::endl;
    }
    
    virtual void layEggs() {
        std::cout << name << " is laying eggs" << std::endl;
    }
    
    virtual void fly() {
        if (canFly) {
            std::cout << name << " is flying with wingspan: " << wingSpan << "m" << std::endl;
        } else {
            std::cout << name << " cannot fly" << std::endl;
        }
    }
    
    double getWingSpan() const { return wingSpan; }
    bool getCanFly() const { return canFly; }
    
    void showInfo() const override {
        Animal::showInfo();
        std::cout << "Wingspan: " << wingSpan << "m, Can Fly: " << (canFly ? "Yes" : "No") << std::endl;
    }
};

// Diamond Problem - Bat sınıfı hem Mammal hem Bird'den kalıtım alır
class ProblematicBat : public Mammal, public Bird {
public:
    ProblematicBat(const std::string& n, int a, double w) 
        : Mammal(n, a, w, 36.5), Bird(n, a, w, 0.3, true) {  // İki kez Animal constructor çağrılır!
        std::cout << "ProblematicBat constructor: " << n << std::endl;
    }
    
    ~ProblematicBat() {
        std::cout << "ProblematicBat destructor" << std::endl;
    }
    
    void makeSound() override {
        std::cout << getName() << " makes ultrasonic sounds" << std::endl;
    }
    
    // Ambiguity problem - hangi showInfo çağrılacak?
    void showInfo() const override {
        std::cout << "=== Problematic Bat Info ===" << std::endl;
        // Mammal::showInfo();  // Bu şekilde belirsizliği çözebiliriz
        // Bird::showInfo();    // Ama hala iki Animal instance var
    }
};

void diamondProblemDemo() {
    std::cout << "\n=== Diamond Problem Demo ===" << std::endl;
    
    // Bu kod derleme hatası verecek çünkü:
    // 1. Animal constructor iki kez çağrılır
    // 2. getName() gibi Animal methods ambiguous
    
    /*
    ProblematicBat bat("Problematic Bat", 2, 0.5);
    
    // Ambiguity errors:
    // bat.getName();        // ERROR: ambiguous
    // bat.showInfo();       // ERROR: ambiguous
    // bat.eat();            // ERROR: ambiguous
    
    // Çözüm - scope resolution kullanmak:
    std::cout << "Name via Mammal: " << bat.Mammal::getName() << std::endl;
    std::cout << "Name via Bird: " << bat.Bird::getName() << std::endl;
    */
    
    std::cout << "ProblematicBat sınıfı Diamond Problem yaratır!" << std::endl;
    std::cout << "- Animal constructor iki kez çağrılır" << std::endl;
    std::cout << "- İki ayrı Animal instance oluşur" << std::endl;
    std::cout << "- Animal methods ambiguous hale gelir" << std::endl;
    std::cout << "- Bellek israfı ve tutarsızlık oluşur" << std::endl;
}

int main() {
    basicMultipleInheritanceDemo();
    diamondProblemDemo();
    return 0;
}
```

### Virtual Inheritance ile Çözüm

```cpp
#include <iostream>
#include <string>
#include <memory>

// Temel sınıf - Animal (aynı)
class Animal {
protected:
    std::string name;
    int age;
    double weight;
    
public:
    Animal(const std::string& n, int a, double w) : name(n), age(a), weight(w) {
        std::cout << "Animal constructor: " << name << std::endl;
    }
    
    virtual ~Animal() {
        std::cout << "Animal destructor: " << name << std::endl;
    }
    
    virtual void eat() {
        std::cout << name << " is eating" << std::endl;
    }
    
    virtual void sleep() {
        std::cout << name << " is sleeping" << std::endl;
    }
    
    virtual void makeSound() = 0;
    
    const std::string& getName() const { return name; }
    int getAge() const { return age; }
    double getWeight() const { return weight; }
    
    virtual void showInfo() const {
        std::cout << "Animal: " << name << ", Age: " << age << ", Weight: " << weight << "kg" << std::endl;
    }
};

// Virtual inheritance ile Mammal
class Mammal : public virtual Animal {  // virtual inheritance
protected:
    double bodyTemperature;
    
public:
    Mammal(const std::string& n, int a, double w, double temp = 37.0) 
        : Animal(n, a, w), bodyTemperature(temp) {
        std::cout << "Mammal constructor: " << name << std::endl;
    }
    
    virtual ~Mammal() {
        std::cout << "Mammal destructor: " << name << std::endl;
    }
    
    virtual void giveBirth() {
        std::cout << name << " is giving birth to live offspring" << std::endl;
    }
    
    virtual void regulateTemperature() {
        std::cout << name << " is regulating body temperature: " << bodyTemperature << "°C" << std::endl;
    }
    
    double getBodyTemperature() const { return bodyTemperature; }
    
    void showMammalInfo() const {
        std::cout << "Mammal traits - Body Temperature: " << bodyTemperature << "°C" << std::endl;
    }
};

// Virtual inheritance ile Bird
class Bird : public virtual Animal {  // virtual inheritance
protected:
    double wingSpan;
    bool canFly;
    
public:
    Bird(const std::string& n, int a, double w, double ws, bool fly = true) 
        : Animal(n, a, w), wingSpan(ws), canFly(fly) {
        std::cout << "Bird constructor: " << name << std::endl;
    }
    
    virtual ~Bird() {
        std::cout << "Bird destructor: " << name << std::endl;
    }
    
    virtual void layEggs() {
        std::cout << name << " is laying eggs" << std::endl;
    }
    
    virtual void fly() {
        if (canFly) {
            std::cout << name << " is flying with wingspan: " << wingSpan << "m" << std::endl;
        } else {
            std::cout << name << " cannot fly" << std::endl;
        }
    }
    
    double getWingSpan() const { return wingSpan; }
    bool getCanFly() const { return canFly; }
    
    void showBirdInfo() const {
        std::cout << "Bird traits - Wingspan: " << wingSpan << "m, Can Fly: " << (canFly ? "Yes" : "No") << std::endl;
    }
};

// Çözülmüş Bat sınıfı - Virtual inheritance kullanarak
class Bat : public Mammal, public Bird {
private:
    double echolocationRange;
    
public:
    // Virtual inheritance'de en türetilmiş sınıf Animal constructor'ı çağırmalı
    Bat(const std::string& n, int a, double w, double echoRange = 100.0) 
        : Animal(n, a, w),  // Sadece bir kez çağrılır
          Mammal(n, a, w, 36.5), 
          Bird(n, a, w, 0.3, true),
          echolocationRange(echoRange) {
        std::cout << "Bat constructor: " << n << std::endl;
    }
    
    ~Bat() {
        std::cout << "Bat destructor: " << getName() << std::endl;
    }
    
    void makeSound() override {
        std::cout << getName() << " makes ultrasonic sounds for echolocation" << std::endl;
    }
    
    void echolocate() {
        std::cout << getName() << " is echolocating with range: " << echolocationRange << "m" << std::endl;
    }
    
    void hibernate() {
        std::cout << getName() << " is hibernating to conserve energy" << std::endl;
    }
    
    double getEcholocationRange() const { return echolocationRange; }
    
    // Artık ambiguity yok - tek Animal instance var
    void showFullInfo() const {
        std::cout << "\n=== " << getName() << " Complete Info ===" << std::endl;
        showInfo();  // Animal::showInfo - ambiguity yok
        showMammalInfo();
        showBirdInfo();
        std::cout << "Echolocation Range: " << echolocationRange << "m" << std::endl;
        std::cout << "================================\n" << std::endl;
    }
};

// Daha karmaşık Virtual Inheritance örneği
class FlyingMammal : public virtual Animal {
protected:
    double maxFlightSpeed;
    
public:
    FlyingMammal(const std::string& n, int a, double w, double speed = 50.0)
        : Animal(n, a, w), maxFlightSpeed(speed) {
        std::cout << "FlyingMammal constructor: " << name << std::endl;
    }
    
    virtual ~FlyingMammal() {
        std::cout << "FlyingMammal destructor: " << name << std::endl;
    }
    
    virtual void glide() {
        std::cout << name << " is gliding at speed: " << maxFlightSpeed << " km/h" << std::endl;
    }
    
    double getMaxFlightSpeed() const { return maxFlightSpeed; }
};

class NocturnalAnimal : public virtual Animal {
protected:
    double nightVisionAccuracy;
    
public:
    NocturnalAnimal(const std::string& n, int a, double w, double vision = 95.0)
        : Animal(n, a, w), nightVisionAccuracy(vision) {
        std::cout << "NocturnalAnimal constructor: " << name << std::endl;
    }
    
    virtual ~NocturnalAnimal() {
        std::cout << "NocturnalAnimal destructor: " << name << std::endl;
    }
    
    virtual void huntAtNight() {
        std::cout << name << " is hunting at night with " << nightVisionAccuracy << "% accuracy" << std::endl;
    }
    
    double getNightVisionAccuracy() const { return nightVisionAccuracy; }
};

// Çoklu virtual inheritance
class AdvancedBat : public FlyingMammal, public NocturnalAnimal, public Mammal {
private:
    std::string species;
    
public:
    AdvancedBat(const std::string& n, int a, double w, const std::string& spec)
        : Animal(n, a, w),  // Tek animal instance
          FlyingMammal(n, a, w, 60.0),
          NocturnalAnimal(n, a, w, 98.0),
          Mammal(n, a, w, 36.0),
          species(spec) {
        std::cout << "AdvancedBat constructor: " << spec << std::endl;
    }
    
    ~AdvancedBat() {
        std::cout << "AdvancedBat destructor: " << species << std::endl;
    }
    
    void makeSound() override {
        std::cout << getName() << " (" << species << ") uses echolocation calls" << std::endl;
    }
    
    void performNightHunt() {
        std::cout << "\n--- " << getName() << " Night Hunt Sequence ---" << std::endl;
        glide();
        huntAtNight();
        makeSound();
        regulateTemperature();
        std::cout << "Hunt completed successfully!\n" << std::endl;
    }
    
    const std::string& getSpecies() const { return species; }
    
    void showAdvancedInfo() const {
        std::cout << "\n=== " << getName() << " (" << species << ") Advanced Info ===" << std::endl;
        showInfo();
        std::cout << "Max Flight Speed: " << getMaxFlightSpeed() << " km/h" << std::endl;
        std::cout << "Night Vision: " << getNightVisionAccuracy() << "%" << std::endl;
        std::cout << "Body Temperature: " << getBodyTemperature() << "°C" << std::endl;
        std::cout << "===============================================\n" << std::endl;
    }
};

void virtualInheritanceDemo() {
    std::cout << "\n=== Virtual Inheritance Solution Demo ===" << std::endl;
    
    // Basit Bat örneği
    std::cout << "\n--- Simple Bat Example ---" << std::endl;
    Bat bat("VampireBat", 3, 0.15, 80.0);
    
    // Artık ambiguity yok!
    std::cout << "Bat name: " << bat.getName() << std::endl;
    std::cout << "Bat age: " << bat.getAge() << std::endl;
    std::cout << "Bat weight: " << bat.getWeight() << "kg" << std::endl;
    
    bat.eat();
    bat.makeSound();
    bat.giveBirth();
    bat.fly();
    bat.echolocate();
    bat.hibernate();
    bat.showFullInfo();
    
    // Gelişmiş Bat örneği
    std::cout << "\n--- Advanced Bat Example ---" << std::endl;
    AdvancedBat advBat("MegaBat", 5, 1.2, "Pteropus vampyrus");
    
    advBat.showAdvancedInfo();
    advBat.performNightHunt();
    
    std::cout << "Virtual inheritance benefits:" << std::endl;
    std::cout << "✓ Tek Animal instance - bellek tasarrufu" << std::endl;
    std::cout << "✓ Ambiguity yok - method çağrıları net" << std::endl;
    std::cout << "✓ Tutarlı veri - tek animal state" << std::endl;
    std::cout << "✓ Doğru constructor/destructor sırası" << std::endl;
}

int main() {
    basicMultipleInheritanceDemo();
    diamondProblemDemo();
    virtualInheritanceDemo();
    return 0;
}
```

## Access Control ve Member Access

### Access Specifiers in Multiple Inheritance

```cpp
#include <iostream>
#include <string>

// Base classes with different access levels
class PublicBase {
public:
    int publicValue;
    
    PublicBase(int val = 10) : publicValue(val) {
        std::cout << "PublicBase constructor: " << publicValue << std::endl;
    }
    
    virtual ~PublicBase() {
        std::cout << "PublicBase destructor" << std::endl;
    }
    
    void publicMethod() {
        std::cout << "PublicBase::publicMethod() - value: " << publicValue << std::endl;
    }
    
protected:
    int protectedValue = 20;
    
    void protectedMethod() {
        std::cout << "PublicBase::protectedMethod() - value: " << protectedValue << std::endl;
    }
    
private:
    int privateValue = 30;
    
    void privateMethod() {
        std::cout << "PublicBase::privateMethod() - value: " << privateValue << std::endl;
    }
};

class ProtectedBase {
public:
    std::string publicString;
    
    ProtectedBase(const std::string& str = "Protected") : publicString(str) {
        std::cout << "ProtectedBase constructor: " << publicString << std::endl;
    }
    
    virtual ~ProtectedBase() {
        std::cout << "ProtectedBase destructor" << std::endl;
    }
    
    void publicStringMethod() {
        std::cout << "ProtectedBase::publicStringMethod() - " << publicString << std::endl;
    }
    
protected:
    std::string protectedString = "ProtectedData";
    
    void protectedStringMethod() {
        std::cout << "ProtectedBase::protectedStringMethod() - " << protectedString << std::endl;
    }
    
private:
    std::string privateString = "PrivateData";
    
    void privateStringMethod() {
        std::cout << "ProtectedBase::privateStringMethod() - " << privateString << std::endl;
    }
};

class PrivateBase {
public:
    double publicDouble;
    
    PrivateBase(double val = 3.14) : publicDouble(val) {
        std::cout << "PrivateBase constructor: " << publicDouble << std::endl;
    }
    
    virtual ~PrivateBase() {
        std::cout << "PrivateBase destructor" << std::endl;
    }
    
    void publicDoubleMethod() {
        std::cout << "PrivateBase::publicDoubleMethod() - " << publicDouble << std::endl;
    }
    
protected:
    double protectedDouble = 2.71;
    
    void protectedDoubleMethod() {
        std::cout << "PrivateBase::protectedDoubleMethod() - " << protectedDouble << std::endl;
    }
    
private:
    double privateDouble = 1.41;
    
    void privateDoubleMethod() {
        std::cout << "PrivateBase::privateDoubleMethod() - " << privateDouble << std::endl;
    }
};

// Multiple inheritance with different access specifiers
class AccessControlDemo : public PublicBase,           // public inheritance
                         protected ProtectedBase,      // protected inheritance
                         private PrivateBase {         // private inheritance
private:
    std::string className;
    
public:
    AccessControlDemo(const std::string& name = "AccessDemo") 
        : PublicBase(100), ProtectedBase("Protected"), PrivateBase(9.99), className(name) {
        std::cout << "AccessControlDemo constructor: " << className << std::endl;
    }
    
    ~AccessControlDemo() {
        std::cout << "AccessControlDemo destructor: " << className << std::endl;
    }
    
    void demonstrateAccess() {
        std::cout << "\n=== Access Control Demonstration ===" << std::endl;
        std::cout << "Class: " << className << std::endl;
        
        // PublicBase access (public inheritance)
        std::cout << "\n--- PublicBase Access (public inheritance) ---" << std::endl;
        std::cout << "publicValue: " << publicValue << std::endl;  // Accessible
        publicMethod();  // Accessible
        
        std::cout << "protectedValue: " << protectedValue << std::endl;  // Accessible in derived
        protectedMethod();  // Accessible in derived
        
        // privateValue not accessible even in derived class
        // std::cout << "privateValue: " << privateValue << std::endl;  // ERROR
        // privateMethod();  // ERROR
        
        // ProtectedBase access (protected inheritance)
        std::cout << "\n--- ProtectedBase Access (protected inheritance) ---" << std::endl;
        std::cout << "publicString: " << publicString << std::endl;  // Accessible
        publicStringMethod();  // Accessible
        
        std::cout << "protectedString: " << protectedString << std::endl;  // Accessible
        protectedStringMethod();  // Accessible
        
        // PrivateBase access (private inheritance)
        std::cout << "\n--- PrivateBase Access (private inheritance) ---" << std::endl;
        std::cout << "publicDouble: " << publicDouble << std::endl;  // Accessible in class
        publicDoubleMethod();  // Accessible in class
        
        std::cout << "protectedDouble: " << protectedDouble << std::endl;  // Accessible in class
        protectedDoubleMethod();  // Accessible in class
    }
    
    // Expose private inheritance members through public interface
    void exposePrivateBasePublic() {
        std::cout << "\n--- Exposing PrivateBase through public interface ---" << std::endl;
        std::cout << "Exposed publicDouble: " << publicDouble << std::endl;
        publicDoubleMethod();
    }
    
    // Using declarations to change access
    using ProtectedBase::publicStringMethod;  // Make it public
    using PrivateBase::publicDoubleMethod;    // Make it public
    
protected:
    // Expose protected members
    using PublicBase::protectedMethod;
    using ProtectedBase::protectedStringMethod;
    
public:
    // Friend function to access private members
    friend void friendAccessTest(const AccessControlDemo& obj);
};

void friendAccessTest(const AccessControlDemo& obj) {
    std::cout << "\n=== Friend Function Access Test ===" << std::endl;
    
    // Can access public members of public inheritance
    std::cout << "Friend accessing publicValue: " << obj.publicValue << std::endl;
    
    // Can access public members of protected inheritance (but not outside class)
    std::cout << "Friend accessing publicString: " << obj.publicString << std::endl;
    
    // Can access public members of private inheritance (but not outside class)
    std::cout << "Friend accessing publicDouble: " << obj.publicDouble << std::endl;
    
    std::cout << "Friend accessing className: " << obj.className << std::endl;
}

void externalAccessTest() {
    std::cout << "\n=== External Access Test ===" << std::endl;
    
    AccessControlDemo demo("ExternalTest");
    
    // PublicBase members accessible (public inheritance)
    std::cout << "External accessing publicValue: " << demo.publicValue << std::endl;
    demo.publicMethod();
    
    // ProtectedBase members NOT accessible externally (protected inheritance)
    // std::cout << "publicString: " << demo.publicString << std::endl;  // ERROR
    // demo.publicStringMethod();  // ERROR (unless using declaration used)
    
    // But using declaration made it accessible
    demo.publicStringMethod();  // OK - using declaration
    
    // PrivateBase members NOT accessible externally (private inheritance)
    // std::cout << "publicDouble: " << demo.publicDouble << std::endl;  // ERROR
    // demo.publicDoubleMethod();  // ERROR (unless using declaration used)
    
    // But using declaration made it accessible
    demo.publicDoubleMethod();  // OK - using declaration
    
    demo.exposePrivateBasePublic();
    
    friendAccessTest(demo);
}

// Complex inheritance hierarchy
class IntermediateClass : public AccessControlDemo {
public:
    IntermediateClass() : AccessControlDemo("Intermediate") {
        std::cout << "IntermediateClass constructor" << std::endl;
    }
    
    ~IntermediateClass() {
        std::cout << "IntermediateClass destructor" << std::endl;
    }
    
    void testInheritedAccess() {
        std::cout << "\n=== Intermediate Class Access Test ===" << std::endl;
        
        // PublicBase members accessible (public inheritance)
        std::cout << "Intermediate accessing publicValue: " << publicValue << std::endl;
        publicMethod();
        protectedMethod();  // Available through using declaration
        
        // ProtectedBase members accessible (protected inheritance in base)
        std::cout << "Intermediate accessing publicString: " << publicString << std::endl;
        publicStringMethod();
        protectedStringMethod();  // Available through using declaration
        
        // PrivateBase members NOT accessible (private inheritance in base)
        // std::cout << "publicDouble: " << publicDouble << std::endl;  // ERROR
        // Can only access through public interface of base class
        exposePrivateBasePublic();
    }
};

class MultipleAccessDemo : public PublicBase, public ProtectedBase {
public:
    MultipleAccessDemo() : PublicBase(200), ProtectedBase("Multiple") {
        std::cout << "MultipleAccessDemo constructor" << std::endl;
    }
    
    ~MultipleAccessDemo() {
        std::cout << "MultipleAccessDemo destructor" << std::endl;
    }
    
    void showAccess() {
        std::cout << "\n=== Multiple Public/Protected Access ===" << std::endl;
        
        // Both bases have public inheritance - all public members accessible
        std::cout << "PublicBase publicValue: " << PublicBase::publicValue << std::endl;
        std::cout << "ProtectedBase publicString: " << ProtectedBase::publicString << std::endl;
        
        PublicBase::publicMethod();
        ProtectedBase::publicStringMethod();
        
        // Protected members accessible in derived class
        PublicBase::protectedMethod();
        ProtectedBase::protectedStringMethod();
    }
};

void accessControlDemo() {
    std::cout << "\n=== Access Control in Multiple Inheritance ===" << std::endl;
    
    AccessControlDemo demo;
    demo.demonstrateAccess();
    
    externalAccessTest();
    
    IntermediateClass intermediate;
    intermediate.testInheritedAccess();
    
    MultipleAccessDemo multiDemo;
    multiDemo.showAccess();
    
    // External access to MultipleAccessDemo
    std::cout << "\nExternal access to MultipleAccessDemo:" << std::endl;
    std::cout << "publicValue: " << multiDemo.PublicBase::publicValue << std::endl;
    std::cout << "publicString: " << multiDemo.ProtectedBase::publicString << std::endl;
    multiDemo.PublicBase::publicMethod();
    multiDemo.ProtectedBase::publicStringMethod();
}

int main() {
    basicMultipleInheritanceDemo();
    diamondProblemDemo();
    virtualInheritanceDemo();
    accessControlDemo();
    
    std::cout << "\n=== Multiple Inheritance Best Practices ===" << std::endl;
    std::cout << "✓ Virtual inheritance kullanarak diamond problem'i önleyin" << std::endl;
    std::cout << "✓ Access specifiers'ı dikkatli seçin" << std::endl;
    std::cout << "✓ using declarations ile access kontrolü yapın" << std::endl;
    std::cout << "✓ Composition'ı multiple inheritance'a tercih edin" << std::endl;
    std::cout << "✓ Interface segregation principle'ı uygulayın" << std::endl;
    std::cout << "✓ Constructor/destructor order'ına dikkat edin" << std::endl;
    std::cout << "✓ Ambiguity'den kaçınmak için scope resolution kullanın" << std::endl;
    
    return 0;
}
```

Çoklu kalıtım, C++'ın güçlü ancak karmaşık özelliklerinden biridir. Diamond problem ve virtual inheritance konularını anlamak, access control mekanizmalarını doğru kullanmak ve composition vs inheritance trade-off'larını değerlendirmek önemlidir. Modern C++'da genellikle composition ve interface-based design tercih edilir.
