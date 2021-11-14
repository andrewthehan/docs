# Chapter 3: Storage and Retrival

---

---

- Databases do two things:
  - Store data ([_"data model"_](./chapter02.md#data-models))
  - Give data back ([_"query language"_](./chapter02.md#query-languages-for-data))

## Storage engines

### Log-structured

Consider the world’s simplest database:

```sh
#!/bin/bash

db_set () {
  echo "$1,$2" >> database
}

db_get () {
  grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

- Key-value store
- `db_set`
  - Appends to a file
  - Has good performance, O(1)
- `db_get`
  - Scans the entire file
  - Has terrible performance, O(n)

#### Index

- Keep some additional metadata on the side which acts as a signpost and helps to locate the wanted data
- Additional structure that is derived from the primary data
- Important trade-off depending on application:
  - Used to optimize _reads_
  - Incurs overhead during _writes_

##### Hash Index

- Simple index for key-value data
- In-memory hash map where every key is mapped to a byte offset in the data file (the location at which the value can be found)
  - Add/update entry in hash map on _write_
  - Use hash map to find offset in the data file on _read_
- Used by Bitcask, the default storage engine in Riak
- Good when there are many writes per key but not many keys

### Page-oriented
