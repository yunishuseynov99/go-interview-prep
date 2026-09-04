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
