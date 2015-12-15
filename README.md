![](https://travis-ci.org/AlexanderDzhoganov/rohr.js.svg?branch=master)

# Rohr.js
## Data validation & transformation library

### Work in progress! [See the test suite for examples](https://github.com/AlexanderDzhoganov/rohr.js/blob/master/test/main.mocha.js)

### Installation

```
npm install rohr
```

To run the tests:
```
npm install -g mocha
npm test
```

## API

Method                          | Description                                                
------------------------------- | ---------------------------------------------------------------------------------------
toPromise(ignoreErrors)         | Returns a promise that resolves to the transformed object, or is rejected with a list of errors
resolve()                       |
object()                        |
error()                         |
prop(propertyName)              | Selects a property in the current scope, adds a validation error if the property is `null` or `undefined`.
optional(propertyName)          | Selects a property in the current scope, no validation if performed if the property exists. Any further chained methods for this property will be ignored if its value is `null` or `undefined`.
set(propertyName, [value/fn])   | Sets the value for a property, value can be a value or a function returning a value/ promise resolving to a value.
value([value/fn])               | Sets the value of the currently selected property (by using `prop()` or `optional()`). Single argument is a value or a function returning a value or a promise resolving to a value.
nuke()                          | Removes the selected property from the object.
scope()                         | Enters the scope of the currently selected property. Only valid if property is an object.
scopeBack()                     | Brings you back to the previous scope.
rootScope()                     | Brings you back to the root scope.
castTo(typeName)                | Casts the currently selected property to a specific type given by `typeName`. Allowed values are: `integer`, `float`, `string`, `date`.
transform(fn)                   | Transforms the currently selected property with a function of the type `function (value) {}`. The function can return a new value or a promise resolving to the new value.
isString()                      | Ensures the currently selected property is of type `String`, adds a validation error otherwise.
isNumber()                      | Ensures the currently selected property is of type `Number`, adds a validation error otherwise.
isDate()                        | Ensures the currently selected property is of type `Date`, adds a validation error otherwise.
isArray()                       | Ensures the currently selected property is of type `Array`, adds a validation error otherwise.
isObject()                      | Ensures the currently selected property is of type `Object`, adds a validation error otherwise.
lookup([array/object], optional:key)     | Performs a look-up for the currently selected property in an array or an object. `key` tells the library which key to look for in the objects if an `array` is passed as the first argument.
map(fn)                         | Maps every object of the currently selected `array` property using a function of the type `function (value) {}`. The function can return a new value for each item in the array or a promise resolving to the new value or a mix of those.
rename(propertyName)            | Renames the currently selected property.
rescope(propertyPath)           | Moves the currently selected property to a new scope. The path starts from the root of the object and accepts dots `.` (e.g. `foo.bar`). Any non-existing properties are created automatically, existing properties are overwritten.
broadcast([propertyName/array]) | Broadcasts the value of the currently selected property to one or more other properties. Single argument is either a string or an array of strings.

### Example:

```es6
var rohr = require('rohr');

var input = {
    foo: "4",
    bar: {
        "yolo": 420,
        "trolo": {
            "bolo": 4
        }
    },
    testId: 1
};

var data = [
    { id: 0, data: 'stuff' },
    { id: 1, data: 'more stuff' },
    { id: 2, data: 'even more stuff' }
]

console.log('IN: ' + JSON.stringify(input, null, 4));

rohr(input)

.prop('foo').isString().castTo('integer').transform(function(val) {
    return val * 2;
})

.prop('bar').isObject().scope()
    .prop('yolo').isNumber().castTo('string').transform(function(val) {
        return 'xxxNoScope' + val + 'xxx';
    })

    .prop('trolo').scope()
        .prop('bolo').isNumber()

.rootScope()

.prop('testId').isNumber().lookup(data, 'id').rescope('more.nested.stuff').scope()
    .prop('data').isString().rename('DATA')

.rootScope()

.toPromise().then(function(obj) {
    console.log('OUT: ' + JSON.stringify(obj, null, 4));
}).catch(function(err) {
    console.log('ERR: ' + JSON.stringify(err, null, 4));
});

```

Result:
```json
IN: {
    "foo": "4",
    "bar": {
        "yolo": 420,
        "trolo": {
            "bolo": 4
        }
    },
    "testId": 1
}
OUT: {
    "foo": 8,
    "bar": {
        "yolo": "xxxNoScope420xxx",
        "trolo": {
            "bolo": 4
        }
    },
    "more": {
        "nested": {
            "stuff": {
                "testId": {
                    "id": 1,
                    "DATA": "more stuff"
                }
            }
        }
    }
}
```

