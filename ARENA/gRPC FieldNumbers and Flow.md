In Protocol Buffers (Protobuf), field numbers (also called tags) are **the unique integer identifiers assigned to each field in a message definition**. Because Protobuf strips away all human-readable field names during serialization to maximize efficiency, the binary wire format relies entirely on these field numbers to identify data.

Here is a breakdown of how field numbers work, their performance implications, and how to manage them safely.

---

## 1. Wire Size & Performance (The 1-Byte Rule)

The size of your serialized message depends heavily on the numbers you choose.
Protobuf encodes the field number alongside its wire type into a combined key:

* **Numbers 1 through 15:** Take only **1 byte** to encode on the wire.
* **Numbers 16 through 2047:** Take **2 bytes** to encode.
* **Higher numbers:** Take **3 or more bytes.**

**Best Practice:** Always reserve field numbers 1 to 15 for your most frequently populated fields to minimize network bandwidth and storage overhead.

---

## 2. Valid Ranges & Constraints

* **Minimum Number:** `1`
* **Maximum Number:** `536,870,911` (`2²⁹ - 1`)
* **Reserved Range:** You cannot use numbers `19000` through `19999`. This range is strictly reserved for the Protocol Buffers compiler (`protoc`) implementation.

---

## 3. The Golden Rule: Never Change or Reuse a Number

Once a `.proto` message type is deployed in production, you must never change or reuse its field numbers.

* Changing a number breaks backward and forward compatibility. Old code reading a new message format will fail to parse the field, treating it as an unknown data block.
* Reusing a deleted field's number causes data corruption. If old data payloads are read by new code, the old data will be incorrectly mapped into your brand-new field.

---

## 4. Retiring Fields Safely Using `reserved`

If you need to delete a field, do not simply erase its line from the file. Another developer might accidentally reuse that number in the future. Instead, delete the field and mark its number (and optionally its name) as `reserved`:

```protobuf
message UserProfile {
  // 2 and 3 were deleted. The compiler will block anyone from using them.
  reserved 2, 3;
  reserved "old_status_code", "temp_id";

  string name = 1;     // 1 byte overhead
  string email = 4;    // 1 byte overhead
  int32 age = 16;      // 2 bytes overhead
}
```

If anyone attempts to declare a field using `2` or `3`, the compiler will throw an error and halt.


===================================================================================================

Almost, but one important correction:

### 1. What do the numbers do?

You write:

```protobuf
message User {
    string name  = 1;
    string email = 2;
    int32 age    = 3;
}
```

The `1`, `2`, `3` are **field numbers/tags**.

When Protobuf serializes this:

```text
name = "John"
email = "john@gmail.com"
age = 27
```

it **doesn't send the field names** `"name"`, `"email"`, `"age"`.

Instead, it sends something conceptually like:

```text
[field #1][value]
[field #2][value]
[field #3][value]
```

So the serialized data is much smaller.

And yes, this is one of the reasons Protobuf is **fast and compact**: you're not repeatedly sending long field names as strings.

But the `.proto` file itself is **not turned into bytecode** by gRPC.

The `.proto` file is used by `protoc` to **generate source code** for your language:

```text
.proto
   ↓
protoc
   ↓
generated Go/C#/Java/etc. code
   ↓
your application
   ↓
Protobuf serialization → compact binary
   ↓
gRPC sends it
```

So **Protobuf = serialization format**, while **gRPC = RPC framework that commonly uses Protobuf**.

---

### 2. What does "unique field numbers" mean?

**Unique within ONE message.**

For example, this is valid:

```protobuf
message User {
    string name = 1;
    string email = 2;
}

message Product {
    string name = 1;
    double price = 2;
}
```

Both messages use `1` and `2`. That's completely fine.

But this is **invalid**:

```protobuf
message User {
    string name  = 1;
    string email = 1; // ❌
}
```

Because inside `User`, `1` would be ambiguous.

Think of the number as the field's **ID inside that particular message type**.

```text
User
 ├── 1 → name
 └── 2 → email

Product
 ├── 1 → name
 └── 2 → price
```

The `1` in `User` and the `1` in `Product` have nothing to do with each other.


==========================================================================================

Think of the whole thing as a pipeline:

```text
.proto file
    ↓
  protoc
    ↓
generated Go code
    ↓
your Go application creates a struct
    ↓
proto.Marshal()
    ↓
binary bytes
    ↓
gRPC sends bytes over network
```

Let's go through each part.

---

## 1. What does `.proto` contain?

It describes your **data structures and services**.

For example:

```protobuf
syntax = "proto3";

package user;

message User {
    string name = 1;
    string email = 2;
    int32 age = 3;
}

service UserService {
    rpc GetUser(GetUserRequest) returns (User);
}

message GetUserRequest {
    int32 id = 1;
}
```

You're basically saying:

> "Here is the structure of my messages, and here are the RPC methods my service provides."

It contains things like:

* `message` definitions
* fields and their numbers
* field types
* `enum`s
* `service`s
* RPC methods
* packages
* imports
* options

It **doesn't contain your actual data**.

---

# 2. What does `protoc` do?

`protoc` is the **Protocol Buffers compiler**.

You give it:

```text
user.proto
```

and it generates Go source code from that definition.

For Go, you typically use:

```bash
protoc --go_out=. --go-grpc_out=. user.proto
```

You get files such as:

```text
user.pb.go
user_grpc.pb.go
```

---

# 3. What is generated in Go?

Your `.proto`:

```protobuf
message User {
    string name = 1;
    string email = 2;
    int32 age = 3;
}
```

causes Go code roughly like this to be generated:

```go
type User struct {
    Name  string
    Email string
    Age   int32
}
```

But the **real generated struct contains additional fields and metadata** used by the Protobuf runtime.

For example, you'll see things conceptually like:

```go
type User struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Name  string `protobuf:"bytes,1,opt,name=name,proto3"`
    Email string `protobuf:"bytes,2,opt,name=email,proto3"`
    Age   int32  `protobuf:"varint,3,opt,name=age,proto3"`
}
```

And `protoc` also generates code describing the message's schema.

For the service:

```protobuf
service UserService {
    rpc GetUser(GetUserRequest) returns (User);
}
```

`protoc` generates Go interfaces/client/server code roughly like:

```go
type UserServiceClient interface {
    GetUser(ctx context.Context, in *GetUserRequest, ...) (*User, error)
}

type UserServiceServer interface {
    GetUser(ctx context.Context, in *GetUserRequest) (*User, error)
}
```

So you don't manually write all the serialization and gRPC plumbing.

---

# 4. What does YOUR application do?

Your application simply works with the generated Go structs.

For example:

```go
user := &User{
    Name:  "John",
    Email: "john@gmail.com",
    Age:   27,
}
```

Your application thinks in terms of normal Go objects/structs.

Then gRPC/Protobuf takes that struct and serializes it.

Conceptually:

```text
Go struct

User{
    Name:  "John",
    Email: "john@gmail.com",
    Age:   27,
}

        ↓

Protobuf serializer

        ↓

binary bytes

        ↓

gRPC

        ↓

network
```

---

# 5. How does serialization become binary?

This is the interesting part.

Suppose:

```go
user := &User{
    Name:  "John",
    Age:   27,
}
```

Your `.proto` says:

```protobuf
message User {
    string name = 1;
    string email = 2;
    int32 age = 3;
}
```

Notice:

```text
name  → 1
email → 2
age   → 3
```

The serializer **doesn't send**:

```text
"name"
"email"
"age"
```

Instead, it sends the **field numbers**.

For `name = "John"`:

```text
field number = 1
wire type    = 2  (length-delimited)
value        = "John"
```

Protobuf combines the field number and wire type:

```text
key = (field_number << 3) | wire_type
```

For `name`:

```text
(1 << 3) | 2
= 8 | 2
= 10
```

So the wire contains:

```text
10 04 4A 6F 68 6E
```

Which means roughly:

```text
10       → field #1 + wire type 2
04       → 4 bytes of data
4A 6F 68 6E → "John"
```

`4A 6F 68 6E` is the UTF-8/ASCII representation of:

```text
J  o  h  n
4A 6F 68 6E
```

For:

```go
Age: 27
```

field number is `3` and `int32` uses the **varint** wire type:

```text
field #3 → wire type 0
```

Key:

```text
(3 << 3) | 0
= 24
```

So the bytes are conceptually:

```text
18 1B
```

where:

```text
18 → field #3 + wire type 0
1B → 27
```

So your message might look approximately like:

```text
0A 04 4A 6F 68 6E 18 1B
```

That's the actual **binary Protobuf payload**.

---

# 6. Why is this fast/small?

Compare JSON:

```json
{
    "name": "John",
    "age": 27
}
```

You repeatedly send:

```text
"name"
"age"
```

and punctuation:

```text
{ } : , "
```

Protobuf instead sends something like:

```text
0A 04 4A 6F 68 6E 18 1B
```

No field names. No JSON punctuation. Numbers are encoded efficiently using things like **varints**.

So:

```text
JSON
  ↓
text representation
  ↓
larger payload

Protobuf
  ↓
compact binary representation
  ↓
smaller payload
```

And because the schema is known ahead of time, the generated code can serialize/deserialize the data efficiently.

---

## The important distinction

Don't think:

> `.proto → bytecode → gRPC`

Think:

```text
                 DEVELOPMENT TIME
                       │
                       ▼
                 user.proto
                       │
                       ▼
                    protoc
                       │
                       ▼
              generated Go code
                       │
                       │
              RUNTIME
                       │
                       ▼
              Your Go application
                       │
                       ▼
                Go struct
                       │
                       ▼
            Protobuf serialization
                       │
                       ▼
                binary bytes
                       │
                       ▼
                    gRPC
                       │
                       ▼
                   network
```

And on the other side:

```text
network
   ↓
gRPC receives bytes
   ↓
Protobuf deserializer
   ↓
generated Go struct
   ↓
server application
```

That's the full picture.
