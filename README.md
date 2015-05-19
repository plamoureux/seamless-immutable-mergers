seamless-immutable-mergers
==========================
This contains a set of custom mergers for the [seamless-immutable](https://github.com/rtfeldman/seamless-immutable) library. It's mainly a showcase for what can be done with custom mergers, but the mergers are hopefully useful on their own.

## The merge API
If you have an immutable object that you want to merge with another object then you do the following in seamless-immutable:

```javascript
var result = immutableObject.merge(otherObject);
```

All properties in `otherObject` will be added to `immutableObject` overwriting any duplicates. Properties in
`immutableObject` that aren't in `otherObject` will be left as is.

### Deep merge
You can also pass in configuration to the merge process. For example using the deep option:

```javascript
var result = immutableObject.merge(otherObject, {deep: true});
```

This will recursively merge all properties that are objects and exists in both the source and target instead of replacing them.

### Custom mergers
You can also pass in a custom merger that overrides the normal merge process. For example:

```javascript
var result = immutableObject.merge(otherObject, {merger: myCustomMerger});
```

A merger is a function that takes the property for the current object, the property for the other object and optionally the config object. Example:

```javascript
function myCustomMerger(current, other, config) {
  return undefined;
}
```

If the merge returns undefined then the merge continues like normal. If the merger returns something than that result is used instead. So checks can be added so a custom merger only operates on specific input, like arrays.

## Supplied mergers
### concatArrayMerger
This is a simple merger that instead of replacing an array with another concats them together. Example:

```javascript
var immutable = require("seamless-immutable");
var mergers = reuqire("seamless-immutable-mergers");

var immutableObject = immutable({
  title: "one",
  items: [1, 2]
});

var otherObject = {
  title: "two",
  items: [3, 4]
};

var result = immutableObject.merge(otherObject, {merger: mergers.concatArrayMerger});
```

The result will be:
```javascript
{
  title: "two",
  items: [1, 2, 3, 4]
}
```

### updatingByIdArrayMerger
This is a merger that operates on arrays that contains objects with specified ids. It tries to merge each object in the target array with the object with the same id from the source array. Example:

```javascript
var immutable = require("seamless-immutable");
var mergers = reuqire("seamless-immutable-mergers");

var immutableObject = immutable({
  array: [
    {
      id: 10,
      status: "ok",
      content: "text"
    }
  ]
});

var otherObject = {
  array: [
    {
      id: 10,
      status: "fail"
    },
    {
      id: 11,
      status: "ok",
      content: "media"
    }
  ]
};

var result = immutableObject.merge(otherObject, {merger: mergers.updatingByIdArrayMerger, mergerObjectIdentifier: "id"});
```

The result will be:
```javascript
{
  array: [
    {
      id: 10,
      status: "fail",
      content: "text"
    },
    {
      id: 11,
      status: "ok",
      content: "media"
    }
  ]
}
```

This merger requires that `mergerObjectIdentifier` is set in the config with the name of the property that identifies the object.
It can be used to update and add to arrays using for example push from the server with only the updated data.

This merger will check both arrays and only do anything if both of them has an object with the specified identifier at position 0. It will then assume that the rest of the arrays only contains such objects.
It can also be used with the `deep` configuration to do this recursively.

