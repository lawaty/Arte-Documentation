# Endpoints Documentation

## Introduction

The Arte framework includes a robust system for defining and managing API endpoints. This document explains how to define endpoints, validate requests, handle responses, and manage routing within the Arte framework.

### 1 Base Class

The core of Arte's endpoint management is the `Endpoint` base class. This class is used to define the initialization and handling logic for each endpoint. Below are the key features provided by the `Endpoint` base class.

### 2 Automated Request Validation

Automated request validation ensures that incoming requests conform to the expected structure. You define this structure in a descriptive manner, and the framework takes care of the rest.

> See docs/validator.md for more information

```php
class AnEndpoint extends Endpoint {
  public function __construct() {
    $exampleExpect = [
      'param1' => [REQ, '/^[a-zA-Z]+$/'],
      'param2' => [REQ, '/^[a-zA-Z]+$/', MANY]
    ];
    $this->init($exampleExpect, $_POST);
  }
}
```

### 3 Automated Request Sanitization

Once a request is validated, you can use it safely in your endpoint's handle method.

```php
public function handle(): Response {
  // Use $this->request here as a sanitized request
}
```

### 4  Clear and powerful Response Object 

The Response object is used to build HTTP responses through a clear and powerful interface. It allows you to add custom HTTP headers and respond with files.

```php
public function handle(): Response {
  // Endpoint processing goes here

  return new Response($response_body, $code, $headers_array);
  // Respond with a file
  return new Response(new ResponseFile(path_to_file));
}
```

### 5 Pretty Routing

After defining your endpoints, add new routes to APP_ROOT/routes.json to map API request URIs to your endpoint definitions. This supports route variables.

```php
{
  "users/{user_id}/friends" => "users/GetFriends.php"
}
```

Firing API_URI/users/10/friends runs the GetFriends endpoint and passes user_id = 10 to the request.

```php
class GetFriends extends Endpoint {
  public function __construct() {
    $this->init([
      'user_id' => [REQ, Regex::ID]
    ]);
  }

  public function handle(): Response {
    // Assuming entity User exists with a getFriends public method
    $user = new User($this->request['user_id']);
    return new Response($user->getFriends());
  }
}
```

### 6 Extendable Base Classes

You can define additional base classes that extend Endpoint for repetitive tasks related to your specific app. For example, if many endpoints require authentication, you can define a class Authenticated that extends Endpoint and includes a pre-handler to authenticate the request.

```php
abstract class Authenticated extends Endpoint {
  protected static array $auth_data;

  protected function init(array $expect, array $request): void {
    // Override parent init to pass additional expectation and data to be validated
    parent::init(
      [...$expect, 'Authorization' => [true, Regex::JWT_BEARER]],
      [...$request, 'Authorization' => getallheaders()['Authorization'] ?? '']
    );

    $this->prehandles[] = function() {
      $data = Authenticator::decode($this->request['Authorization']);
      // Process data here
      if(!authorized)
        return new Response("Not authorized", 401);

      $this->auth_data = $data;
    };
  }
}
```

### 7 Automated Logger

The framework logs all requests to API_ROOT/logs by date.

### 8 Automated Tests

Following the AAA (Arrange, Act, Assert) testing approach, you can arrange endpoint test scenarios. The framework will run them and provide you with the results.

```php
class TestableEndpoints extends Endpoint implements Testable {
  public function arrange(): array {
    $input_generator1 = function() {
      return array to be used as request payload simulation;
    };
    $scenario1 = new Scenario('scenario_identifier', $input_generator1);
    $checker1 = function($input, $output) {
      return true if assertion is true and false otherwise;
    };
    $assertions = [
      new Assertion('assertion1', $checker1),
      // Add more assertions as needed
    ];
    $scenario1->addAssertion(...$assertions);

    // Repeat to create more scenarios

    return [$scenario1, $scenario2, etc...];
  }
}
```

Call API/test/endpoint_route and pass an optional trials_count. The Endpoint base class will apply all test runs and provide you with the results, including which scenarios failed due to which assertions and all other test run information.

## Exception Handling
TODO