# What is Arte

Arte is an `API-centric` framework that provides the essential features for hosting APIs. This emphasizes the framework's vision to completely separate backend logic from from potential front-end views clients adhering to the SOA fundamentals to facilitate scalalbility and maintainability.

#### What Arte excels at ?
Arte perfectly suits database management systems by helping you organize your system and takes over many of the repetitive tasks automatically while maintaining max performance by `eliminating additional computational overhead`. That's done through utilizing framework core utilities as they are designed to `decrease db calls, and intensive computational operations` as much as possible.

Another thing that is worth mentioning is that Arte is light-weight and built in native php so you can **integrate it with other existing frameworks** without affecting performance, or the complexity of your system in case you need to benefit from its features or project organized structure without HTTP serve. 

## Getting Started
Contact lawaty@drolez-apps.cloud to obtain a licensed copy of the framework.

## Arte Architecture
Arte is designed as a combination of `SOA` and `Multi-tier` architectures as it breaks functionalities into separate endpoints and also encapsulates model layer to be separated from business services flow. 

#### Arte System Components
Arte consists of `core` and `app`. Arte core contains all framework core components, utilities, base classes, etc... They are designed to handle repetitive tasks optimally. Arte app contains all app-related stuff and that helps developers work directly on their business logic without worrying about the underlying framework mechanics. 

#### Layered Architecture
Arte systems are written in two main components which are `model` and `endpoints` layers. model layer encapuslate system entities, database mappers, and services apart from their usage or endpoints flow. Endpoints use model classes to deliver the services final output.

# Traversing Arte in a top-down approach

### 1. Defining Endpoints
> See docs/endpoints.md for more details

Arte core defines a base class named `Endpoint` that is used to define endpoint initialization and handler. It provides you with so many features to automate many processes while keeping performance and maintainability in mind.
1. `Automated Request Validation`: Just tell the endpoint initializer what is your expected request structure in a descriptive manner and the request method, and the framework will validate it. (have a look at `docs/validator.md` for more about validation utilities)
```php
class AnEndpoint extend Endpoint {
  public function __construct() {
    $exampleExpect = [
      'key1' => [REQ, '/^[a-zA-Z]+$/'],
      'key2' => [REQ, '/^[a-zA-Z]+$/', MANY]
    ];
    $this->init($exampleExpect, $_POST);
  }
}
```

2. `Automated Request Sanitizer`: You can use the sanitized request securely in your endpoint handle method
```php
public function handle(): Response {
  Use $this->request here as a sanitized request
}
```

3. `Clear and Powerful Response Object`: Response object is used to build the http response through clear and powerful interface. Allows you to add custom http headers and also allows responding with files:
```php
public function handle(): Response {
  endpoint processing goes here

  return new Response($response_body, $code, $headers_array);
}
```

4. `Pretty Routing`: After you define your endpoints in the endpoints directory, add new route to APP_ROOT/routes.json that `maps an api request uri to your endpoint definition path relative to endpoints directory`. This also supports route variables
```json
{
  'users/create' => "users/Create.php" // a file where Create class (which is an Endpoint class) is defined
}
```

### 2. Connecting Arte to a database
Follow ROOT_DIR/.env.example to define db connection creds at ROOT_DIR/.env that can either be SQLite or MySQL DB.

### 3. Arte DB Automation

> Have a look at `docs/db-automation.md` for more

This can be useful in case you need to easier deployment process by running an `automation script` that automatically creates/inserts default records for you with a single script run. You can define your db deployment operations in a descriptive manner at `app/model/database` and then run `API_URL/db-script` and it will do the magic for you.

In addition to deployment, db-script also generates a `relation map` at app/model/database/joins.json every run that can be used with Arte `CustomMapper` base class to automatically join mappers together without writing complex SQL Queries by yourself for improved performance and readability.

### 4. Database Mappers

> Have a look at `docs/mappers.md` for more

in app/model/mappers, you can define mappers for your system entities. Mappers correspond to your db tables in 1:1 correspondance and must be named as "<entity_name_here>Mapper". Mapper objects are coupled with Entity objects so that it can provide with you with db auto-sync as you are going to see later. Mapper base class provide you with the following capabilities:
1. Specify db table mapped information for strict integrity checking.
```php
class Entity1Mapper extends Mapper {
  public static string $table = 'tablename';
  public static array $required = ['field1', 'field2'];
  public static array $optional = ['field3', 'field4'];
}
```
2. auto-sanitize in and out data by ensuring that required data are all available while entity creation and fetch as well.
```php
$entity1 = Mapper1::create($data); // throws RequiredPropertyNotFound if not all required data are available and return object of type Entity1
```
3. Powerful and abstract CRUD utilities
```php
$entity1 = Mapper1::get($conditons); // conditions is an associative array of filters and returns the first match
$all_matches = Mapper1::getAll($conditions); // returns Entities object of all matches. Entities will be discussed later on but it is a data structure just like arrays but with automated, and optimized entity object creation.
/**
 * @var Entity $entity
 */
foreach($all_matches as $entity)
  $entity->get('field1', 'field2'); // process fetched entities as you want

$entity1->delete(); // delegates the call to the coupled mapper
$entity1->set('a_key', 'a_new_value'); // changes entity property to the new value and stores changed values to be ready for save alter
$entity1->save(); // delegates the call to the coupled mapper to save all changes to db
``` 

### 5. Entities

> Have a look at `docs/entities.md` for more

In app/model/entities, you can define your business entities by extending Entity base class defined inside Arte core. Entity base class provides you with the following:
1. The ability to `define your entity properties` just as any object for clearer interfacing with the entity everywhere.
```php
class Entity1 extends Entity {
  int $property1;
  string $property2 = 'default_value';
  Entity2 $foreign_property;
}
```

2. `Secure` and `Powerful` setters and getters with `strict property checking` to always ensure your entities' integrity for you. Powerful means that you can pass any type of values to your properties and it will parse it if needed. Also Entities allow automatic foreign linking meaning that if an entity contains a foreign_id for another entity for example, you can define this relation in your entity class and you can then get this
```php
class Entity2 extends Entity {
  ...properties_here
}

$entity1 = new Entity1($id);
$entity1->set('foreign_property', $entity2_id);
$entity2_in_entity1 = $entity1->get('foreign_property'); // returns Entity2 object
```

3. Implements ArrayAccess so that you can access your objects like arrays for better readability and simplicity In addition to JsonSerialize, and Comparison operators
```php
$entity1['property']; // the same as $entity->get('property')
$entity1['property'] = 'new_value'; // the same as $entity->set('property', 'new_value');
json_encode($entity1); // returns a json string for all properties inside the object
$entity1 == $entity2; // ensures that they are of the same type and all properties are identical
```

4. `auto-sync with db`. This allows you to directly access entity properties without the need to explicitly fetch them from database. Entity utilizes its coupling with Mapper base class to automatically fetch its data from the database whenever you need it. It is worth mentioning that it is greatly optimized to decrease db calls using memoization techniques.

```php
$entity = new Entity1(20); // 20 is the id of the desired object in the database
$entity->get('property1'); // automatically loads the object information from db if not loaded and that reduces code complexity and won't fire a db call until you actually need it so that it ensures that no useless db calls are executed.
```
### 6. Defining business services
define classes as you want at app/model/services that interact with all the autoloaded classes in arte app or core and use these services in your endpoints.

# How Arte Works ?

The program execution starts in the `index.php` file. This file works in conjunction with instances of Logger, Router, and Endpoint to deliver the response.

The `$response->echo()` function is responsible for closing the connection with the client and flushing the output buffer.

The `Router::route()` method searches for the requested endpoint by the client and returns an instance of the `Endpoint` class if found. This allows us to call run method on the Endpoint that run all prehandles and the main handler for this endpoint.


### How Router Works ?

The Router component plays a crucial role in the framework by parsing the incoming requests and locating the desired endpoint. Here's an overview of how the Router works:

1. The Router receives the incoming request from the client.
2. It analyzes the request and extracts relevant information such as the HTTP method and the requested endpoint URL.
3. The Router then searches for the requested endpoint inside routes.json and returns and endpoint instance holding request payload. Here the Router passes the responsibility to the caller and he is free to use the returned endpoint instance then.

Router only works when you use Arte to serve REST endpoints but it is also great at organizing system components in general.

### Integrating Arte with other frameworks
You can require autoload.php whenever you need to use Arte classes and you can programmatically run endpoints as if they are normal controllers without HTTP access.

# More Reading ?

#### More about endpoints ?
See docs/endpoints.md

#### More about writing app models ?
See docs/entities.md
See docs/mappers.md

#### Utilities
See docs/helpers.md

#### Arte Plugins
See docs/plugins.md

#### More about Automation in Arte ?
See docs/automation.md

#### More about System Stability ?
See docs/No-Failure Guarantee.md
