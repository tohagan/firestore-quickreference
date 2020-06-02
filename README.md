# FireStore & FireStorage Security Language - Quick Reference

## FireStore Resource

`resource`                      = Requested document
`resource['__name__']`          = Document path
`resource.id`                   = Document key
  - e.g. `resource['__name__'] == /databases/(default)/documents/collection/$(resource.id)`
`resource.data.{field}`         = Current field values (field => value)
`request.resource.data.{field}` = Pending document state after operation

## FireStore Request

`request.auth`
`request.auth.uid`
`request.auth.token.email`
`request.auth.token.email_verified`
`request.auth.token.phone_number`
`request.auth.token.name`
`request.auth.token.firebase.identities`
`request.auth.token.firebase.sign_in_provider`
`request.{get|list|create|update|delete}`  = Operation type
`request.path`  = Resource path
`request.query.{limit|offset|orderBy}`  = Optional Query
   - e.g. `allow list: if request.query.limit <= 50`
`request.time`  = Time of request
  - e.g. `request.time == request.resource.data.updateAt`

## Types

- `String`, `Integer`, `Float`, `Boolean`, `Bytes`
- `Number` = `Integer` or `Float`
- `Duration`, `Timestamp`
- `LatLng`
- `List`, `Map`, `MapDiff`, `Set`
- `Path`, `Request`, `Response`

## Conditions and Errors

Error values don't stop computation of conditions:

```js
error && true	  => error
error && false	=> false
error || true	  => true
error || false	=> error
```

## Namespace Functions

- `debug(value)` - Returns `value`. Use in conditions. Outputs to `firestore-debug.log` in Emulator only.

- `maths.abs()/.ceil()/.floor()/.isInfinite()/.isNan()/.pow()/.round()/.sqrt()`,

- `timestamp.date(year, month, day): Timestamp`
- `timestamp.value(epoch): Timestamp`
- `duration.time(hours, mins, secs, nanos)`
- `duration.value(integer, unit)` unit is w=weeks,d=days,h=hours,m=minutes,s=seconds,ms=millisecs,ns=nanosecs
- `latlng.value(lat, lng): LatLng`, `LatLng.distance(): Float`, `LatLng.latitude():Float`, `LatLng.longiture():Float`
- `hashing.crc32(bytes)/.csc32c(bytes)/.md5(bytes)/.sha256(bytes)`
- `hashing.crc32(string)/.csc32c(string)/.md5(string)/.sha256(string)`

## Classes
- String:
  e.g. `string(true) == "true"`, `string(1) == "1"`, `string(2.0) == "2.0"`, `string(null) == "null"`
  - `str.lower()`, `str.upper()`,
  - `str.matches(re)`  - re = Google RE Syntax
  - `str.replace(re, sub)`
  - `str.size()`, `str.split()`, `str.trim()`
  - `str.toUtf8(): Bytes`

- Timestamp
  - `timestamp + duration`
  - `timestamp - duration`
  - `duration + timestamp`
  - `timestamp.date()/.time()`
  - `timestamp.year()/.month()/.day()`
  - `timestamp.hours()/.minutes()/.seconds()/.nanos()`
  - `timestamp.toMillis()/.dayofYear()`
  - `timestamp.dayofWeek()/.dayofYear()`

- Duration
  - `timestamp - timestamp`
  - `duration + duration`
  - `duration - duration`
  - Use Namespace functions to create a Duration

- List:
  - e.g. `['a', 'b']`
  - `list.hasAll(list): Set`
  - `list.hasAny(list): Boolean`
  - `list.hasOnly(list): Boolean`
  - `list.join(separator): String`
  - `list.size(): Integer`
  - `list.toSet(): Set`

- Set:
  - e.g. `['a', 'b'].toSet()` - How to create a Set
  - `set.difference(set): Set`
  - `set.hasAll(list): Set`
  - `set.hasAny(list): Boolean`
  - `set.hasOnly(list): Boolean`
  - `set.intersection(set): Set`
  - `set.union(set): Set`
  - `set.size(set): Integer`
- Map:
  - `map.get(key, default): Value`
  - `map.keys(): List`
  - `map.values(): List`
  - `map.size(): Integer`
  - `map.diff(map): MapDiff` - used to validate changes at the field level
    - `MapDiff.affectedKeys/addedKeys().changedKeys()/.removedKeys()/.unchangedKeys()`
    - e.g. `request.resource.data.diff(resource.data).affectedKeys().hasOnly(["a"]);`
        = was only field 'a' added/removed/updated ?

## FireStore only functions:

- `exists(path: Path)` - Does document exist?
  - e.g. `allow write: if exists(/databases/$(database)/documents/things/other)`
- `existsAfter(path: Path)` - Will document exist after this operation?
  - same as `getAfter(path) != null`
- `get(path: Path)` - Get Document content
- `getAfter(path: Path)` - Get what the document would be after this operation. Useful in batch writes.

## Storage API Request / Response

`request.time: Timestamp`  = Time of request

`request.resource.name`  = Full file name (including path)
`request.resource.bucket`
`request.resource.metadata`
`request.resource.size`  = File size in bytes
`request.resource.contentType`

`resource.name: String`  = Full file name (including path)
`resource.bucket: String`
`resource.generation`  - Object generation. Used for object versioning.
`resource.metageneration`  - Object generation. Used for object versioning.
`resource.size: Integer`  = File size in bytes
`resource.timeCreated: Timestamp`
`resource.updated: Timestamp`
`resource.md5Hash: String`
`resource.crc32c: String`
`resource.etag`
`resource.contentDisposition`
`resource.contentEncoding`
`resource.contentLanguage`
`resource.contentType`
`resource.metadata: Map<String, String>` - Developer provided fields


```js
// Allow a read if the file was created less than one hour ago
allow read: if request.time < resource.timeCreated + duration.value(1, 'h');
```

## Storage Rules

```js
// Specify the Storage service with $bucket name
service firebase.storage {
  match /b/{bucket}/o {
    ...
  }
}
```

```js
allow read;
allow write;
allow read, write;
allow read: if <condition>;
allow write: if <condition>;

// Allow read at "path/to/object" if condition evaluates to true
match /path {
  match /to {
    match /object {
      allow read: if <condition>;
    }
  }
}
```

```js
// Allow read at any path "/path/*/newPath/*", if condition evaluates to true
match /path/{str1} {  // Binds str1: String
  match /newPath/{str2} { // Binds str2: String
    // Matches "path/to/newPath/newObject" or "path/from/newPath/oldObject"
    allow read: if <condition>;
  }
}
```

```js
// Allow read at any path "/**", if condition evaluates to true
match /{path1=**} { // Binds path1: Path
  // Matches anything at or below this, from "path", "path/to", "path/to/object", ...
  allow read: if <condition>;
}
```
