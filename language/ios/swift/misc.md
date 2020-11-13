# Misc


## How to check one variable's type

```swift
let string = "Hello"
let stringArray = ["one", "two"]
let dictionary = ["key": 2]

print(type(of: string)) // "String"

// Get type name as a string
String(describing: type(of: string)) // "String"
String(describing: type(of: stringArray)) // "Array<String>"
String(describing: type(of: dictionary)) // "Dictionary<String, Int>"

// Get full type as a string
String(reflecting: type(of: string)) // "Swift.String"
String(reflecting: type(of: stringArray)) // "Swift.Array<Swift.String>"
String(reflecting: type(of: dictionary)) // "Swift.Dictionary<Swift.String, Swift.Int>"

```

## How to handle type change
