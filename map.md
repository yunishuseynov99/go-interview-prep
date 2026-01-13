golang website info

- **map** in golang is an implementation of a hash table

==map[KeyType]ValueType

-KeyType may be any type that is comparable

-Map types are reference types

== var m map[string]int

-the value of m above is nil; it doesn’t point to an initialized map. 
-nil map behaves like an empty map when reading, 
-writing to a nil map will cause a runtime panic

== m = make(map[string]int)

-The make function allocates and initializes a hash map data structure and returns a map value that points to it

== i := m["route"]

-If key doesn’t exist, we get the value type’s zero value. In this case the value type is int, so the zero value is 0:


= n := len(m)

-len function returns on the number of items in a map:

= delete(m, "route")

-delete function removes an entry from the map

-if key doesn’t exist, delete function doesn’t return anything, and will do nothing.

=i, ok := m["route"]

-If key doesn’t exist, i is the value type’s zero value (0). Second value (ok) is a bool (true if the key exists , and false if not).




-To iterate over the contents of a map, use the range keyword:

= for key, value := range m {
    fmt.Println("Key:", key, "Value:", value)
}

-To initialize a map with some data, use a map literal:

= commits := map[string]int{
    "rsc": 3711,
    "r":   2138,
    "gri": 1908,
    "adg": 912,
}

-The same syntax may be used to initialize an empty map, which is functionally identical to using the make function:

= m = map[string]int{}

-Maps are not safe for concurrent use

-If you need to read from and write to a map from concurrently executing goroutines, the accesses must be mediated by some kind of synchronization mechanism. 
-common ways to protect maps is with sync.RWMutex. or sync.Map

-iteration order of a map is not specified and is not guaranteed to be the same from one iteration to the next.

===================================================================================================================================================================
<About buckets and some more stuff> link(https://phati-sawant.medium.com/internals-of-map-in-golang-33db6e25b3f8)

-Internally a structure called map header is created and the variable m receives a pointer to this structure, the map header contains all the meta information about the map, like:

•The number of entries that are currently in the map
•The number of buckets in a map is always equal to power of two hence the log(buckets) stored to keep the value small
•Pointer to the bucket array that is stored in contiguous memory location,
•Hash seed which is random to create each map differently

-Each bucket stores an array of hash codes, a list of key value pairs and an overflow pointer. 
The array of hash code stores hash code for each key in the bucket, this is used for faster comparison. 
Each bucket can store maximum of 8 such key value pairs. 
The overflow pointer can point to a new bucket if more that 8 values are received by a bucket.

-?What happens when map grows?
-Every time the number of elements in a bucket reaches a certain limit, i.e the load factor which is 6.5, the map will grow in size by doubling the number of buckets. 
This is done by creating a new bucket array consisting of twice the number of buckets than the old array, 
and then copying all the buckets from old to new array but this is done very efficiently over time and not at once.
During this the map maintains a pointer to the old bucket which is stored at the end of the new bucket array.

-?What happens when we insert a new value in the map?
= m[“green”] = “#00ff00”
-Hash function is called and a hash code is generated for the given key, 
based on a part of the hash code a bucket is determined to store the key value pair. 
Once the bucket is selected the entry needs to be stored in that bucket. 
The complete hash code of the incoming key is compared with all the hashes from the initial array of hash codes i.e h1, h2, h3…. 
if no hash code matches that means this is a new entry. 
Now if the bucket contains an empty slot then the new entry is stored at the end of the list of key value pairs, 
else a new bucket is created and the entry is stored in the new bucket and the overflow pointer of old bucket points to this new bucket.


some key points{
•In golang maps are internally array of buckets
•The lookup time for map is O(1)
•You can modify a map while iterating on it
•Map iteration is random
•The load factor for maps is 6.5
•The number of entries in each bucket is 8
•The number of bucket always doubles
•Overflow pointer is used to point to a new bucket when a already full bucket receives a new value
}

