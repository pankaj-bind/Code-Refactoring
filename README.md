## Flow of solution
<img width="1549" height="862" alt="image" src="https://github.com/user-attachments/assets/6e1e0b9c-e3cc-4f45-85f6-c7187fb7bb44" />



### Manual Calculation

To understand what the tools are doing, you can also calculate the metrics manually. The CK suite includes the following six metrics:

#### 1. Weighted Methods per Class (WMC)
* **What it measures:** The complexity of a class.
* **How to calculate:** It's the sum of the complexities of all methods within that class. In its simplest form, each method is given a complexity of 1, so the WMC is just the **total number of methods** in the class. A high WMC can indicate that a class has too many responsibilities.

#### 2. Depth of Inheritance Tree (DIT)
* **What it measures:** How far down a class is in the inheritance hierarchy.
* **How to calculate:** It's the length of the path from the class to the root class in the inheritance tree. A root class has a DIT of 0. Deeper trees can lead to greater design complexity, but also promote more code reuse.

#### 3. Number of Children (NOC)
* **What it measures:** The number of immediate subclasses of a class.
* **How to calculate:** Simply **count the number of classes** that directly inherit from the class in question. A high NOC can indicate a well-reused abstraction, but it can also mean that an abstraction is being misused.

#### 4. Coupling Between Object classes (CBO)
* **What it measures:** The degree of coupling a class has with other classes.
* **How to calculate:** It's a **count of the other classes** to which a class is coupled. A class is coupled to another if it uses the methods or instance variables of that other class. High coupling is undesirable as it complicates modification and testing.

#### 5. Response For a Class (RFC)
* **What it measures:** The potential number of methods that can be executed in response to a message received by an object of that class.
* **How to calculate:** It's the **number of methods in the class** plus the **number of remote methods** directly called by the methods of that class. A high RFC suggests high complexity and can make a class difficult to test.

#### 6. Lack of Cohesion in Methods (LCOM)
* **What it measures:** The degree to which methods within a class are related to each other.
* **How to calculate:** This is the most complex metric with several variations. The original idea is to look at pairs of methods in a class. If two methods use at least one common instance variable, they are considered cohesive. LCOM is a count of the pairs of methods that are **not cohesive** minus the pairs that are. A high LCOM value indicates the class is trying to do too many different things and should probably be split.

### General Thresholds for CK Metrics

Here are some commonly cited upper-bound thresholds. If a class exceeds these values, it's a "red flag" that warrants an investigation.

| Metric | Threshold (Upper Bound) | What it Might Indicate if Exceeded |
| :--- | :--- | :--- |
| **WMC** | > 20 | The class is too complex and may have too many responsibilities (violates SRP). |
| **DIT** | > 5 | The inheritance hierarchy is getting too deep, which can make it hard to understand and maintain. |
| **NOC** | > 7 | The base class is becoming fragile. A change to it could impact a large number of subclasses. |
| **CBO** | > 14 | The class is highly coupled to other classes, making it difficult to change or reuse independently. |
| **RFC** | > 50 | The class has a complex interaction with other parts of the system, making it difficult to test and debug. |
| **LCOM** | > 1 (for LCOM4) or Low Cohesion | The class lacks a clear, single purpose and should likely be split into smaller, more cohesive classes. |


### Bad Code Example (High Metrics)

This code features a `MegaManager` class that handles users, products, and notifications, leading to low cohesion and high complexity. It also has a very deep inheritance chain for `Level5Manager`.


```cpp
// ---- (Other classes this manager is coupled to) ----
#include <iostream>
#include <vector>
#include <string>

class Database { public: void connect() {} };
class Emailer { public: void send(std::string msg) {} };
class Logger { public: void log(std::string msg) {} };
class User { public: std::string name; };
class Product { public: std::string sku; };


// ---- (Deep Inheritance Chain) ----
class Entity { public: int id; };
class Person : public Entity { public: std::string name; };
class Employee : public Person { public: int employeeId; };
class Manager : public Employee { public: int reportCount; };
class Level5Manager : public Manager { public: void approveMegaProject() {} }; // DIT is high


// ---- (The "God Class") ----
class MegaManager {
private:
    Database db;
    Emailer mailer;
    Logger logger;
    std::vector<User> users;
    std::vector<Product> products;

public:
    // User Management Methods
    void addUser(User u) {
        db.connect();
        users.push_back(u);
        logger.log("User added");
    }
    void removeUser(int userId) {
        logger.log("User removed");
    }

    // Product Management Methods
    void addProduct(Product p) {
        db.connect();
        products.push_back(p);
        logger.log("Product added");
    }
    void updateProductStock(int productId) {
        logger.log("Stock updated");
    }

    // Notification Methods
    void notifyAdmin(std::string message) {
        mailer.send(message);
    }
};
```

-----

### Refactored Code Example (Low Metrics)

This version breaks up the `MegaManager` into smaller, cohesive classes (`UserManager`, `ProductManager`, `NotificationService`). The deep inheritance is replaced with a flatter structure and composition.

```cpp
#include <iostream>
#include <vector>
#include <string>

// ---- (Dependencies remain the same) ----
class Database { public: void connect() {} };
class Emailer { public: void send(std::string msg) {} };
class Logger { public: void log(std::string msg) {} };


// ---- (Flatter Inheritance and Composition) ----
class Person {
public:
    int id;
    std::string name;
};

enum Role { Staff, Manager, SeniorManager };

class Employee { // No deep inheritance
public:
    Person profile;
    int employeeId;
    Role role;
    void approveProject() {
        if (role >= Manager) { /* logic */ }
    }
};


// ---- ("God Class" is broken into smaller, focused classes) ----
class UserManager {
private:
    Database& db;
    Logger& logger;
    std::vector<User> users;

public:
    UserManager(Database& d, Logger& l) : db(d), logger(l) {}
    void addUser(User u) {
        db.connect();
        users.push_back(u);
        logger.log("User added");
    }
    void removeUser(int userId) {
        logger.log("User removed");
    }
};

class ProductManager {
private:
    Database& db;
    Logger& logger;
    std::vector<Product> products;

public:
    ProductManager(Database& d, Logger& l) : db(d), logger(l) {}
    void addProduct(Product p) {
        db.connect();
        products.push_back(p);
        logger.log("Product added");
    }
    void updateProductStock(int productId) {
        logger.log("Stock updated");
    }
};

class NotificationService {
private:
    Emailer& mailer;
public:
    NotificationService(Emailer& e) : mailer(e) {}
    void notifyAdmin(std::string message) {
        mailer.send(message);
    }
};
```

-----

### Comparison Table

This table shows the dramatic improvement in the key metrics after refactoring.

| Metric | "Bad" Code (Class) | Value | "Good" Code (Class) | Value | Improvement |
| :--- | :--- | :--- | :--- | :--- |:--- |
| **WMC** | `MegaManager` | **5** | `UserManager` | 2 | **Reduced complexity** |
| | | | `ProductManager` | 2 | |
| **DIT** | `Level5Manager` | **5** | `Employee` | **1** | **Flattened hierarchy** (via Composition) |
| **CBO** | `MegaManager` | **5** | `UserManager` | 2 | **Reduced coupling** |
| | | | `ProductManager` | 2 | |
| | | | `NotificationService` | 1 | |
| **RFC** | `MegaManager` | **\~8** | `UserManager` | \~4 | **Smaller, testable units** |
| | | | `ProductManager` | \~4 | |
| **LCOM** | `MegaManager` | **High** | `UserManager` | **Low** | **Increased cohesion** (Single Responsibility) |
