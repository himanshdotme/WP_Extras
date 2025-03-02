# PHP Object-Oriented Programming: Traits, Singletons, Abstract Classes, and Interfaces

### Traits in PHP
Traits are a mechanism for code reuse in single inheritance languages like PHP. Traits allow you to reuse methods across different classes without using inheritance.

#### Key Characteristics of Traits:
- Introduced in PHP 5.4
- Cannot be instantiated on their own
- Used to declare methods that can be used in multiple classes
- Reduces code duplication
- Solves some limitations of single inheritance

Example of a Trait:
```
<?php
// Define a trait
trait Logger {
    protected $logFile = 'application.log';
    
    public function log($message) {
        $timestamp = date('Y-m-d H:i:s');
        file_put_contents(
            $this->logFile, 
            "[$timestamp] $message" . PHP_EOL, 
            FILE_APPEND
        );
    }
    
    public function setLogFile($filename) {
        $this->logFile = $filename;
    }
}

// Class using the trait
class User {
    use Logger;
    
    private $name;
    
    public function __construct($name) {
        $this->name = $name;
        $this->log("User {$this->name} was created");
    }
    
    public function login() {
        $this->log("User {$this->name} logged in");
    }
}

// Another class using the same trait
class Product {
    use Logger;
    
    private $name;
    private $price;
    
    public function __construct($name, $price) {
        $this->name = $name;
        $this->price = $price;
        $this->setLogFile('products.log'); // Override default log file
        $this->log("Product {$this->name} was created with price {$this->price}");
    }
}

// Usage
$user = new User("John");
$user->login();

$product = new Product("Laptop", 999);

```
---
### Singleton Pattern in PHP
The Singleton pattern ensures a class has only one instance and provides a global point of access to it.

#### Key Characteristics of Singletons:
- Private/protected constructor to prevent direct instantiation
- Private/protected clone method to prevent cloning
- Private/protected wakeup method to prevent deserialization
- Static instance property to hold the single instance
- Static method to get the instance

Example of a Singleton:
```
<?php
class Database {
    // Static property to hold the single instance
    private static $instance = null;
    
    private $connection;
    
    // Private constructor to prevent direct instantiation
    private function __construct() {
        $this->connection = new PDO('mysql:host=localhost;dbname=myapp', 'username', 'password');
        echo "Database connection established\n";
    }
    
    // Private clone method to prevent cloning
    private function __clone() {}
    
    // Private wakeup method to prevent deserialization
    private function __wakeup() {}
    
    // Public static method to get the instance
    public static function getInstance() {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    public function query($sql) {
        return $this->connection->query($sql);
    }
}

// Usage
$db1 = Database::getInstance(); // Outputs: "Database connection established"
$db2 = Database::getInstance(); // No output because it reuses the same instance

var_dump($db1 === $db2); // bool(true) - both variables reference the same instance
```
### Singleton Trait Implementation
You can also implement a singleton pattern using a trait for reusability:
```
<?php
trait SingletonTrait {
    private static $instance = null;
    
    private function __construct() {}
    private function __clone() {}
    private function __wakeup() {}
    
    public static function getInstance() {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}

class Config {
    use SingletonTrait;
    
    private $settings = [];
    
    public function load($file) {
        $this->settings = json_decode(file_get_contents($file), true);
    }
    
    public function get($key) {
        return $this->settings[$key] ?? null;
    }
}

// Usage
$config = Config::getInstance();
$config->load('config.json');

$anotherConfig = Config::getInstance(); // Same instance
```
### Abstract Classes in PHP
Abstract classes are classes that cannot be instantiated and are designed to be extended by other classes.

#### Key Characteristics of Abstract Classes:
- Cannot be instantiated directly
- May contain abstract methods (methods without implementation)
- May contain concrete methods (methods with implementation)
- Subclasses must implement all abstract methods
- Used for creating a common base class with shared functionality

Example of an Abstract Class:
```
<?php
abstract class Shape {
    protected $color;
    
    public function __construct($color = 'red') {
        $this->color = $color;
    }
    
    // Concrete method with implementation
    public function getColor() {
        return $this->color;
    }
    
    // Abstract method without implementation
    abstract public function calculateArea();
    
    // Another abstract method
    abstract public function calculatePerimeter();
}

class Circle extends Shape {
    private $radius;
    
    public function __construct($radius, $color = 'red') {
        parent::__construct($color);
        $this->radius = $radius;
    }
    
    // Implementation of abstract method
    public function calculateArea() {
        return pi() * $this->radius * $this->radius;
    }
    
    // Implementation of abstract method
    public function calculatePerimeter() {
        return 2 * pi() * $this->radius;
    }
}

class Rectangle extends Shape {
    private $width;
    private $height;
    
    public function __construct($width, $height, $color = 'blue') {
        parent::__construct($color);
        $this->width = $width;
        $this->height = $height;
    }
    
    public function calculateArea() {
        return $this->width * $this->height;
    }
    
    public function calculatePerimeter() {
        return 2 * ($this->width + $this->height);
    }
}

// Usage
$circle = new Circle(5, 'green');
echo "Circle area: " . $circle->calculateArea() . "\n";
echo "Circle color: " . $circle->getColor() . "\n";

$rectangle = new Rectangle(4, 6);
echo "Rectangle area: " . $rectangle->calculateArea() . "\n";
echo "Rectangle color: " . $rectangle->getColor() . "\n";

// The following would cause an error:
// $shape = new Shape(); // Error: Cannot instantiate abstract class
```
### Interfaces in PHP
Interfaces define a contract that implementing classes must adhere to.

#### Key Characteristics of Interfaces:
- Cannot be instantiated
- Contain only method declarations, no implementations
- Cannot contain properties (only constants)
- Classes can implement multiple interfaces
- All interface methods must be implemented by the class

Example of Interfaces:
```
<?php
interface Drawable {
    public function draw();
}

interface Resizable {
    public function resize($percentage);
    public function getSize();
}

class Square implements Drawable, Resizable {
    private $side;
    
    public function __construct($side) {
        $this->side = $side;
    }
    
    public function draw() {
        echo "Drawing a square with side {$this->side}\n";
    }
    
    public function resize($percentage) {
        $this->side = $this->side * $percentage / 100;
    }
    
    public function getSize() {
        return $this->side * $this->side; // Area
    }
}

class Text implements Drawable {
    private $content;
    
    public function __construct($content) {
        $this->content = $content;
    }
    
    public function draw() {
        echo "Drawing text: {$this->content}\n";
    }
}

// Function that accepts any drawable object
function renderObject(Drawable $object) {
    $object->draw();
}

// Usage
$square = new Square(5);
$text = new Text("Hello World");

renderObject($square); // "Drawing a square with side 5"
renderObject($text);   // "Drawing text: Hello World"

$square->resize(200);  // Double the size
echo "New square size: " . $square->getSize() . "\n";
```

### Comparing and Combining These Concepts
#### Abstract Classes vs. Interfaces:


| Abstract Classes                            | 	Interfaces                               |
|---------------------------------------------|-------------------------------------------|
| Can have method implementations             | 	Only method declarations                 |
| Can have properties	                        | Only constants                            |
| A class can extend only one abstract class	 | A class can implement multiple interfaces |
| Can enforce a base structure	               | Define a contract of capabilities         |

#### Real-world Example Combining All Concepts:

```
<?php
// Singleton trait for reusable singleton pattern
trait SingletonTrait {
    private static $instance = null;
    
    private function __construct() {}
    private function __clone() {}
    private function __wakeup() {}
    
    public static function getInstance() {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}

// Logger trait for reusable logging functionality
trait LoggerTrait {
    protected function log($message) {
        $class = get_class($this);
        $timestamp = date('Y-m-d H:i:s');
        echo "[$timestamp][$class] $message\n";
    }
}

// Cacheable interface defines methods for objects that can be cached
interface Cacheable {
    public function getCacheKey();
    public function getCacheData();
    public function loadFromCacheData(array $data);
}

// DatabaseObject interface for objects that can be saved to database
interface DatabaseObject {
    public function save();
    public function load($id);
    public function delete();
}

// Abstract base for all data models
abstract class Model implements Cacheable, DatabaseObject {
    use LoggerTrait;
    
    protected $id;
    protected $created;
    protected $modified;
    
    public function __construct($id = null) {
        if ($id) {
            $this->load($id);
        } else {
            $this->created = time();
            $this->modified = time();
        }
    }
    
    public function save() {
        $this->modified = time();
        $this->log("Saving " . get_class($this) . " with ID {$this->id}");
        // Database save logic would go here
    }
    
    public function delete() {
        $this->log("Deleting " . get_class($this) . " with ID {$this->id}");
        // Database delete logic would go here
    }
    
    // Common implementation for Cacheable interface
    public function getCacheKey() {
        return get_class($this) . ':' . $this->id;
    }
    
    // Common implementation for Cacheable interface
    public function getCacheData() {
        return get_object_vars($this);
    }
    
    // Abstract method that child classes must implement
    abstract protected function validate();
}

// Concrete implementation example
class User extends Model {
    private $username;
    private $email;
    private $password;
    
    public function setUsername($username) {
        $this->username = $username;
    }
    
    public function setEmail($email) {
        $this->email = $email;
    }
    
    public function setPassword($password) {
        $this->password = password_hash($password, PASSWORD_DEFAULT);
    }
    
    public function load($id) {
        $this->log("Loading User with ID $id");
        // In real application, would load from database
        $this->id = $id;
        $this->username = "user$id";
        $this->email = "user$id@example.com";
        $this->created = time() - 3600; // 1 hour ago
        $this->modified = time() - 1800; // 30 minutes ago
    }
    
    public function loadFromCacheData(array $data) {
        foreach ($data as $key => $value) {
            if (property_exists($this, $key)) {
                $this->$key = $value;
            }
        }
    }
    
    protected function validate() {
        if (empty($this->username)) {
            throw new \Exception("Username cannot be empty");
        }
        if (!filter_var($this->email, FILTER_VALIDATE_EMAIL)) {
            throw new \Exception("Invalid email address");
        }
        return true;
    }
}

// Cache manager as a singleton
class CacheManager {
    use SingletonTrait;
    
    private $cache = [];
    
    public function store(Cacheable $object) {
        $this->cache[$object->getCacheKey()] = $object->getCacheData();
    }
    
    public function retrieve($key) {
        return $this->cache[$key] ?? null;
    }
    
    public function has($key) {
        return isset($this->cache[$key]);
    }
    
    public function clear($key = null) {
        if ($key === null) {
            $this->cache = [];
        } else {
            unset($this->cache[$key]);
        }
    }
}

// Usage example
try {
    // Create and configure a user
    $user = new User();
    $user->setUsername("john_doe");
    $user->setEmail("john@example.com");
    $user->setPassword("secret123");
    $user->save();
    
    // Cache the user
    $cache = CacheManager::getInstance();
    $cache->store($user);
    
    // Load another user
    $anotherUser = new User(2);
    
    // Prove singleton works
    $anotherCache = CacheManager::getInstance();
    var_dump($cache === $anotherCache); // true
    
} catch (\Exception $e) {
    echo "Error: " . $e->getMessage();
}
```