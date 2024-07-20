# Arte Entity

The `Entity` class is a crucial component of our framework, representing an abstract entity in the system. It provides various methods and properties to manage and manipulate data entities, ensuring synchronization with the database and supporting complex interactions.

## Overview

The `Entity` class implements the `JsonSerializable` and `ArrayAccess` interfaces, enabling it to be easily serialized to JSON and accessed like an array. This class serves as a base class for all entities in the system, providing core functionality that can be extended by derived classes.

## Key Features

- **Simplified Data Management**: The `Mapper` class provides a consistent interface for performing CRUD (Create, Read, Update, Delete) operations. Methods like `create()`, `save()`, `get()`, and `delete()` offer a streamlined approach to managing entity records in the database.
- **Automated Record Handling**: When using the `Mapper` class, entities can leverage methods like `save()` and `delete()` to automatically handle record updates and deletions. The `save()` method updates existing records based on changes, while `delete()` removes records, simplifying database management tasks.
- **Dynamic Property Handling**: It provides methods like `naiveGet()` and `set()` to dynamically manage properties, including handling foreign key relationships and data sanitization. This dynamic handling allows for flexible and extensible data managemen.
- **Automatic Change Tracking**: The `Entity` class includes mechanisms for tracking changes made to its properties (`$changes`). This feature is crucial for operations that need to persist only modified data, reducing unnecessary database interactions.
- **Data Loading and Synchronization**: The `load()` method allows for the bulk loading of data into the entity, while the `isLoaded()` and `setLoaded()` methods manage the synchronization state. This ensures that entities are properly synchronized with their data source before performing operations.
- **Property Protection**: The `$protected` property allows for defining which properties should be protected from direct access or modifications. This feature is useful for enforcing security and data integrity.
- **Foreign Key Handling**: The `$foreigns` property allows for mapping and managing foreign key relationships, enabling the easy association of entities with other domain objects. This feature simplifies the management of complex relationships between entities.

- **Handy access**: Implementing `ArrayAccess` and `JsonSerializable` interfaces made it look like simple arrays. 

## Usage

#### Example1:

```php
class User extends Entity
{
    protected string $name;
    protected string $email;

    public function __construct($id)
    {
        parent::__construct($id);
    }
}

$user = new User(1);
$user->load(['name' => 'John Doe', 'email' => 'john@example.com']);
echo json_encode($user); // {"id":1,"name":"John Doe","email":"john@example.com"}
```

#### Example2: (with nested entities)

```php
class Order extends Entity
{
    protected string $orderNumber;
    protected array $items;

    public function __construct($id)
    {
        parent::__construct($id);
    }
}

class Item extends Entity
{
    protected string $itemName;
    protected float $price;

    public function __construct($id)
    {
        parent::__construct($id);
    }
}

$order = new Order(1);
$order->load([
    'orderNumber' => 'ORD123',
    'items' => [
        new Item(1), new Item(2)
    ]
]);

foreach ($order->items as $item) {
    $item->load(['itemName' => 'Product 1', 'price' => 19.99]);
}
echo json_encode($order);
```

## Exception Handling
TODO