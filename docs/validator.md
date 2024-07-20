# Validator Service Documentation

### Introduction

The Validator class provides a robust system for validating complex data structures in PHP. It allows you to define expectations for your data, validate incoming data against these expectations, and filter out invalid data, all while providing clear error messages when validation fails.

### Features

- **Nested Validation**: Supports complex nested data structures.
- **Flexible Criteria**: Criteria can be regular expressions, callable functions, or nested validators.
- **Automated Filtering**: Filters valid data automatically.
- **Detailed Error Messages**: Provides detailed paths to invalid data.

## How To Use

### 1. Defining Validation Criteria

The Validator class uses an array to define the structure and validation criteria for the input data. Each key in the array corresponds to an expected parameter, and the value is an array with three elements: required, is_many, and validation.

- **required**: defines whether this paramter is required `REQ` or not `OPT`
- **validation**: The validation criteria, which can be a regular expression, a callable function, or another Validator instance.
- **is_many**: if the parameter can be multiple `MANY` or single `SINGLE`.

#### Example: 
```php
$expect = [
  'key1' => [REQ, '/^[a-zA-Z]+$/'],
  'key2' => [REQ, '/^[a-zA-Z]+$/', MANY],
  'key3' => [OPT, '/^[a-zA-Z]+$/'],
  'key4' => [OPT, '/^[a-zA-Z]+$/', MANY],
  'key5' => [REQ, [
    'key1' => [REQ, '/^[a-zA-Z]+$/'],
    'key2' => [REQ, '/^[a-zA-Z]+$/', MANY],
    'key3' => [OPT, '/^[a-zA-Z]+$/'],
    'key4' => [OPT, '/^[a-zA-Z]+$/', many]
  ]],
  'key6' => [REQ, [
    'key1' => [REQ, '/^[a-zA-Z]+$/'],
    'key2' => [REQ, '/^[a-zA-Z]+$/', MANY],
    'key3' => [OPT, '/^[a-zA-Z]+$/'],
    'key4' => [OPT, '/^[a-zA-Z]+$/', many]
  ], MANY],
  'key7' => [OPT, [
    'key1' => [REQ, '/^[a-zA-Z]+$/'],
    'key2' => [REQ, '/^[a-zA-Z]+$/', MANY],
    'key3' => [OPT, '/^[a-zA-Z]+$/'],
    'key4' => [OPT, '/^[a-zA-Z]+$/', many]
  ]],
  'key8' => [OPT,, MANY [
    'key1' => [REQ, '/^[a-zA-Z]+$/'],
    'key2' => [REQ, '/^[a-zA-Z]+$/', MANY],
    'key3' => [OPT, '/^[a-zA-Z]+$/'],
    'key4' => [OPT, '/^[a-zA-Z]+$/', many]
  ]],
];
```

### 2. Creating A validator instance

Create a Validator instance by passing the defined expectations array to the constructor.

```php
$validator = new Validator($expect);
```

### 3. Validatng Data

Validate the input data by calling the validate method. This method returns null if the data is valid, or a string indicating the path to the invalid data.

```php
$input = [
  'key1' => 'John',
  'key2' => ['Doe', 'Smith'],
  'key3' => 'Alice',
  'key4' => ['Bob', 'Carol'],
  'key5' => [
    'subkey1' => 'David',
    'subkey2' => ['Eve', 'Frank']
  ],
  'key6' => [
    [
      'subkey1' => 'Judy',
      'subkey2' => ['Ken', 'Leo']
    ],
    [
      'subkey1' => 'Peggy',
      'subkey2' => ['Quentin', 'Rob']
    ]
  ]
];

$result = $validator->validate($input);
echo $result ?? 'All inputs are valid.';

```

### 4. Filtering Valid Data

Retrieve the filtered valid data using the getFiltered method.

```php
$filteredData = $validator->getFiltered();
print_r($filteredData);
```

### 5. Throwing Detailed Errors

Use the throw method to throw a detailed error when invalid data is detected.

```php
if ($result) {
  $validator->throw($result, $input);
}
```

### 6. Printing the Validation Tree

You can print the entire validation tree for debugging purposes using the printTree method.

```php
$validator->printTree();
```

## Extent of Validation

The Validator class can validate highly complex data structures, including deeply nested arrays and objects. It can handle:

- Simple data types with regular expressions.
- Complex nested data structures with nested Validator instances.
- Callable functions for custom validation logic.
- Multiple instances of a parameter with the MANY flag.

#### Example

```php
$complexExpect = [
  'users' => [REQ, MANY, [
    'id' => [REQ, SINGLE, '/^\d+$/'],
    'name' => [REQ, SINGLE, '/^[a-zA-Z]+$/'],
    'friends' => [OPT, MANY, [
      'id' => [REQ, SINGLE, '/^\d+$/'],
      'name' => [REQ, SINGLE, '/^[a-zA-Z]+$/']
    ]]
  ]],
  'metadata' => [OPT, SINGLE, [
    'timestamp' => [REQ, SINGLE, '/^\d+$/'],
    'source' => [OPT, SINGLE, '/^[a-zA-Z]+$/']
  ]]
];

$validator = new Validator($complexExpect);
$input = [
  'users' => [
    [
      'id' => '123',
      'name' => 'John',
      'friends' => [
        ['id' => '456', 'name' => 'Doe'],
        ['id' => '789', 'name' => 'Smith']
      ]
    ],
    [
      'id' => '124',
      'name' => 'Jane'
    ]
  ],
  'metadata' => [
    'timestamp' => '1627889182'
  ]
];

$result = $validator->validate($input);
echo $result ?? 'All inputs are valid.';
$filteredData = $validator->getFiltered();
print_r($filteredData);
```

