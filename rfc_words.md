
# Improved error handling

Note - this RFC depends on having 'out parameters' similar to [C#'s implementation](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/out-parameter-modifier).


## Introduction

Error handling is one of the gnarliest problems in programming. In particular, error handling around IO leads to great debates about whether something should be an exception or whether it should be an error.

Sometimes you want an error, sometimes you want an exception. Imagine two functions that are identical, but have different semantic meaning:

```php
// Create a data file in a new directory 
// $directoryName should be a unique directory name that
// does not currently exist
function createUniqueFile($directoryName, $text) {
    mkdir($directoryName, 0755, true);
    file_put_contents("foo.txt", $text);
}

// Update a data file in a directory that may already exist
function updateData($directoryName, $text) {
    mkdir($directoryName, 0755, true);
    file_put_contents("foo.txt", $text);
}
```

For the function `createUniqueFile`, the failure of mkdir to create a directory due to it already existing is an error condition that means continuing execution of the function is not possible.

For the function `updateData`, the failure of mkdir to create a directory is not a problem, and it is fine for execution to continue.

Some languages solve this dilemma by over-using exceptions. Other languages do not support type specific exceptions, and instead all errors are return.

```go
stringValue, err := Hello("Hiya")
```

This RFC proposes a syntax for the calling code to determine if the calling code wants to handle the error locally, or if an exception should be thrown.

## Proposal

Through something like out-parameters, allow 


```
function foo($input, out ValidationException $exceptionOutParameter): ?int {

    $validationProblems = validateInput($input);
    
    // No problems, process data
    if (count($validationProblems) === 0) {
       return bar($input);
    }
    
    $exception = new ValidationException($validationProblems);
    
    // Check if out parameter was used
    if (is_void($exceptionOutParameter) === true) {
        //it was not passed in, so throw exception
        throw $exception; 
    }

    // out parameter was used, so don't throw exception
    // as calling user wants to handle the error with 
    // normal code flow.
    $exceptionOutParameter = $error;
}  
```

```php
$result = foo($input, $error);
````

Any error in foo() will lead to the error being stored in the $error variable.

```php
$result = foo($input);
```
Any error in foo() will lead to an exception being thrown.


This allows the calling code to decide whether the called code should throw an exception, or handle the error in normal code flow.


## Spare words

### Exact exceptions to be used

The exact exceptions to be associated with errors/warnings generated but internal code will be determined in a separate RFC. There are approximately 2700 errors/warnings/notices that need to be looked at. It is too much work to do as part of an RFC that might not pass.

The general rules should be followed:

i) Specific exceptions are good.

ii) Avoid complex hierarchies.

e.g. avoid trying to make a base 'IO exception' and that is used by mkdir, fopen, and other file related functions, instead have separate MkdirException, FopenException.
