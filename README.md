# FireStore & FireStorage Security Rules - Quick Reference

Firebase developers are JavaScript coders, you'll likely already be very familier with the meaning of the most of the functions and method names available in FireStore and StoreStorage security rules language. I found the official reference a bit slow to use so I created this condensed Quick Reference as a faster way to look up methods, properties and function names and as reminder of constructs and patterns.

It's still a work in progress.  Feel free to contribute or fix but please retain the compact format. I think it really needs some "best practice" pattern examples - perhaps on a separate page.

NOTE: This is NOT an official Firebase reference and may be incorrect or out of date. By using this quick reference you agree that it is solely your responsability to confirm the validity and correctness of this information.

## Official References

**Firestore**

- [Guide: Firebase Security Rules](https://firebase.google.com/docs/rules)
- [Reference: Firestore Security Rules](https://firebase.google.com/docs/reference/rules/rules)
- [Blog Jun 17, 2020: Firestore Security Rules Improvements](https://firebase.googleblog.com/2020/06/new-firestore-security-rules-features.html)
- [Release Notes](https://firebase.google.com/support/release-notes/security-rules)

**Storage**

- [Guide: Storage Rules](https://firebase.google.com/docs/storage/security)
- [Reference: Storage Rules](https://firebase.google.com/docs/reference/security/storage)

**Tech Support**

- [Firestore Google Group](https://groups.google.com/g/google-cloud-firestore-discuss)

## Other Resources

- [Firestore Security Rules Cookbook](https://fireship.io/snippets/firestore-rules-recipes/)
- [What does it mean that “Firestore security rules are not filters”?](https://medium.com/firebase-developers/what-does-it-mean-that-firestore-security-rules-are-not-filters-68ec14f3d003)
* [The top 10 things to know about Firestore when choosing a database for your app](https://medium.com/firebase-developers/the-top-10-things-to-know-about-firestore-when-choosing-a-database-for-your-app-a3b71b80d979)

## Videos

[![Security Rules! 🔑 | Get to know Cloud Firestore #6](https://img.youtube.com/vi/eW5MdE3ZcAw/0.jpg)](https://www.youtube.com/watch?v=eW5MdE3ZcAw "Security Rules! 🔑 | Get to know Cloud Firestore #6")

[![Unit testing security rules with the Firebase Emulator Suite](https://img.youtube.com/vi/VDulvfBpzZE/0.jpg)](https://www.youtube.com/watch?v=VDulvfBpzZE "Unit testing security rules with the Firebase Emulator Suite")

[![Introduction to Firebase security rules - Firecasts](https://img.youtube.com/vi/QEuu9X9L-MU/0.jpg)](https://www.youtube.com/watch?v=QEuu9X9L-MU "Introduction to Firebase security rules - Firecasts")


## Security Rules

```js
[rules_version = <<version>>]
service <<service>> {
  // Match the resource path.
  match <<path>> {
    // Allow the request if the following conditions are true.
    allow <<methods>> : if <<condition>>
  }
}
```

- ``<<version>>`` ::= ``'2'``
- ``<<service>>`` ::= ``cloud.firestore`` |  ``cloud.firestorage``
- ``<<path>>``  ::= database or storage location 
- `<<methods>>` ::= `get` | `list` | `create` | `update` | `delete` | `read` | `write`
- `<<condition>>` ::== A condition based on `request` & `resource` objects and bound variables from `<<path>>`.

## Permissions Methods

- `allow get` = Allow a single document read
- `allow list` = Allow collection reads and queries `db.collection("users").orderBy("name")`
- `allow read` = Allow `get` and `list`
- `allow create` - Allow `docRef.set()` or `collectionRef.add()`
- `allow update` - Allow `docRef.update()` or `docRef.set()`
- `allow delete` - Allow `docRef.delete()`
- `allow write` - Allow `create`, `update` or `delete`.

## Conditions and Errors

Error values don't stop computation of conditions:

```js
error && true => error
error && false	=> false
error || true	=> true
error || false	=> error
```
## Ternary 

According to [this post](https://stackoverflow.com/questions/60217594/is-there-a-way-to-make-the-ternary-operator-work-for-cloud-firestore-security-ru), the ternary expression has now been added.

`<<boolean condition>>` ? `<<true-value>>` : `<<false-value>>`

## FireStore Validation

- `request` = auth, time, path and create/update/delete data for this operation
- `resource` =  Document BEFORE this change
- `request.resource` = Document AFTER this change ... if it were successful!

## FireStore Resource

- `resource`                      = Requested document
- `resource['__name__']`          = Document path
- `resource.id`                   = Document key
  - e.g. `resource['__name__'] == /databases/(default)/documents/collection/$(resource.id)`
- `resource.data.{field}`         = Current field values (field => value)
- `request.resource.data.{field}` = Pending document state after operation

## FireStore Request

- `request.resource.data` - field values AFTER the requested opertion if successful.
- `request.auth`
- `request.auth.uid`
- `request.auth.token.email`
- `request.auth.token.email_verified`
- `request.auth.token.phone_number`
- `request.auth.token.name`
- `request.auth.token.firebase.identities`
- `request.auth.token.firebase.sign_in_provider`
- `request.{get|list|create|update|delete}`  = Operation type
- `request.path`  = Resource path
- `request.query.{limit|offset|orderBy}`  = Optional Query
   - e.g. `allow list: if request.query.limit <= 50`
- `request.time`  = Time of request
  - e.g. `request.time == request.resource.data.updateAt`

## Custom Claims

- Sets values in `request.auth.token.<key>`
- [Control Access with Custom Claims and Security Rules](https://firebase.google.com/docs/auth/admin/custom-claims)

Reserved claim keys: 
- `amr`, `at_hash`, `aud	`, `auth_time`, `azp`, `c_hash	`, `cnf`, `exp`, `firebase`, `iat`, `iss`, `jti`, `nbf`, `nonce`, `sub`

## Types

- `String`, `Integer`, `Float`, `Boolean`, `Bytes`
- `Number` = `Integer` or `Float`
- `Duration`, `Timestamp`
- `LatLng`
- `List`, `Map`, `MapDiff`, `Set`
- `Path`, `Request`, `Response`

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

## Storage API - Request

- `request.time: Timestamp` - Time of request
- `request.resource.name`  - Full file name (including path)
- `request.resource.bucket` - Cloud Storage Buckct 
- `request.resource.metadata` - Map of custom properties
- `request.resource.size`  - File size in bytes
- `request.resource.contentType` - MIME content type (e.g 'image/jpg')

## Storage API - Resource

- `resource.name: String`  = Full file name (including path)
- `resource.bucket: String` = Google Cloud Storage bucket
- `resource.size: Integer`  = File size in bytes
- `resource.generation`  - Object generation. Used for object versioning.
- `resource.metageneration`  - Object generation. Used for object versioning.
- `resource.timeCreated: Timestamp` - create at
- `resource.updated: Timestamp` - last update at
- `resource.md5Hash: String` - MD5 Hash
- `resource.crc32c: String`  - CRC-32C Hash

Headers sent when downloading the file:

- `resource.etag` - [ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag)
- `resource.cachecontrol` - [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
- `resource.contentDisposition` -  [Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)
- `resource.contentEncoding` - [Content-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding) of the file (for example 'gzip')
- `resource.contentLanguage` - [Content-Language](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language) of the file content (e.g. 'en' or 'es')
- `resource.contentType` - MIME [Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) (e.g 'image/jpg')
- `resource.metadata: Map<String, String>` - Developer provided fields

## Storage API - Samples

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
