### Proposal

<!---
Brief introduction
--->
At least three base collection types can be considered as prevalent in modern-day languages: lists, sets and maps.
GraphQL specification already defines a [list](http://facebook.github.io/graphql/October2016/#sec-Lists) collection type that can also serve as a set.
This proposal aims to fill in the gap and add the missing collection type - the map.

<!---
Motivation
--->
The map collection type would benefit the GraphQL specification for the following reasons:
1. A standardized exchange of map collections. At the moment, developers are forced to implement in-house solutions that:
    1. cannot take full advantage of GraphQL specification features such as [selection sets](http://facebook.github.io/graphql/October2016/#sec-Selection-Sets), [non-null field](http://facebook.github.io/graphql/October2016/#sec-Input-Types) or [introspection](http://facebook.github.io/graphql/October2016/#sec-Introspection) and results in obscured GraphQL schema;
    2. promotes special case scenarios in server and client implementations to handle (de)serialization of the map type which is a standard data type in many modern languages;
2. The map collection type simplifies the transfer of arbitrary JSON structures in special case scenarios avoiding "JSON structure as String" situations (please read the third point).
3. A good practice should be encouraged by a convention rather than enforced by a limitation!

<!---
The more formal spec goes here
--->
### Example Map specification

A GraphQL map is a special collection type of unique key-value pairs which declares the type of each key and value in the map (referred to as the _key type_ and _value type_ of the map respectively). Each key-value pair in the map must be identified by exactly one key. The key type must be either String or ID and the value type is not restricted.

Map key-value pairs are serialized in any order. Each key in the map is serialized as a String and each value is serialized as per the value type. To denote that a field uses a Map type the key and the value types are passed as map type parameters like this: `pets: Map(String, Pet)`, where String is the key type and Pet is the value type.

#### Input Coercion
<a name="input_coercion"></a>

GraphQL servers must return an ordered list of key-value pairs as the result of a map type. Each key-value pair in the map must be the result of a result coercion of the corresponding key and value types. If a reasonable coercion is not possible they must raise a field error. In particular, if a non‐map is returned, the coercion should fail, as this indicates a mismatch in expectations between the type system and the implementation.

#### Output Coercion

When expected as an input, key-value pairs are accepted only when each key-value pair in the map can be accepted by the map's key and value types.

If the key-value pair passed as an input to a map type is not a map and not the null value, it should be coerced as though the input was a map of size one, where the key-value pair passed is the only key-value pair in the map. This is to allow inputs that accept a “var-args” to declare their input type as a map; if only one argument is passed (a common case), the client can just pass that key-value pair rather than constructing the map.

#### JSON representation

The given declaration of `pets: Map(String, Pet)` with two example key-value pairs ``{("pet1", "Chewy"), ("pet2", "Oscar")}`` translates to a JSON Object with a list of two key-value pairs:

```JSON
pets: {
  "pet1": "Chewy",
  "pet2": "Oscar"
}
```

<!---
Additional remarks
--->
### Other considerations

#### Support for Map various implementations

As specified in the [Input Coercion](#input_coercion) section, the map type is serialized to an ordered list of key-value pairs. This retains the semantics of various Map implementations such as unordered, ordered and sorted maps.

#### Limiting Key type to String
The above specification limits the key type to String. The decision is taken due to the following reasons:
1. JSON specification supports only String keys; scuh key-value pairs are native to JSON objects;
