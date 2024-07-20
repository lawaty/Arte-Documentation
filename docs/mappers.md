# Arte Mappers

The `Mapper` base class serves as a crucial component in managing database interactions for entities in a system. It abstracts the complexities of database operations and provides a clean interface for performing CRUD (Create, Read, Update, Delete) operations. Here's a comprehensive look at its features from a high-level perspective.

## Overview

The `Mapper` class is designed to handle the persistence layer of an application. It provides methods for managing the lifecycle of entities, including creating, updating, retrieving, and deleting records in a database. The class uses a combination of static and instance methods to interact with the database and manage entity state.

## Features

1. **Simple Mapper Definitions**: You can define mapper-to-persistance mapping through simple data structures:

    1.1. `$entity_type`: Represents the type of the entity associated with the mapper. It is `dynamically set` based on the class name and helps in identifying the specific entity type being managed.

    1.2. `$table`: Specifies the database table associated with the entity. This property `must be defined in subclasses` to ensure correct table interactions.

    1.3. `$required`: An array of required properties for the entity. These properties `must be present` when creating or updating records.

    1.4. `$optional`: An array of optional properties that can be included in entity records. These properties are `not mandatory` but can be present in records.

2. **Clear and Powerful CRUD Interface**: 

    1.1. `create(array $entity_data)`: Creates a new record in the database using the provided data. Returns a fully constructed entity instance if the creation is successful.

    1.2. `get($filters = [])`: Retrieves a single entity based on the provided filters. Supports numeric IDs and array-based filters.    
    
    1.3. `getAll($filters = [])`: Retrieves multiple entities based on filters. Returns an instance of `Entities` containing all matching records. Have a look at docs/helpers.md to know more about `Entities` Container.

    1.4. `save()`: Saves the current state of the entity. Updates the record if changes are present.

3. **Data Sanitization**: By providing a `sanitize` method, the Mapper class ensures that data is correctly formatted and validated before insertion into the database. This helps in maintaining data integrity and consistency.

4. **Entity Management**: The `Mapper` class supports the full lifecycle of an entity, from creation and retrieval to updates and deletion. This comprehensive support simplifies the development of applications that require database interactions.

## Example Usage

```php
class UserMapper extends Mapper
{
    public static string $table = 'users';
    public static array $required = ['name', 'email'];
    public static array $optional = ['phone'];
}

class User extends Entity
{
    protected string $name;
    protected string $email;
    protected ?string $phone = null;

    public function __construct($id)
    {
        parent::__construct($id);
    }
}

// Create a new user
$user_data = ['name' => 'John Doe', 'email' => 'john.doe@example.com'];
$user = UserMapper::create($user_data);

// Update an existing user
$user->set('phone', '123-456-7890');
$user->save();

// Retrieve a user
$retrieved_user = UserMapper::get(['id' => 1]);

// Delete a user
$retrieved_user->delete();
```

## Exception Handling
TODO