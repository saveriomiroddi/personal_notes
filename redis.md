# Redis

## Commands

`<pattern>` is a full match; it can use `*`.

| Type    | Command                              | Comment                                                                           |
| :------ | :----------------------------------- | --------------------------------------------------------------------------------- |
| General | `KEYS <pattern>`                     | list keys                                                                         |
| General | `GET <key>`                          | get a key's value                                                                 |
| General | `DEL <key>`                          | delete a key; returns the number of keys deleted                                  |
| General | `FLUSHALL`                           | clear/purge all the keys                                                          |
| Hash    | `HGET`                               | get a key                                                                         |
| Hash    | `HGETALL`                            | get all the fields/values                                                         |
| Hash    | `HMSET`                              | set multiple fields/values      (ruby library: use mapped_hmset for hash sources) |
| Hash    | `HSET`                               | set a field/value                                                                 |
| Hash    | `HSETNX <key> <field> <value>`       | set a field/value, if the field doesn't exist. auto creates keys.                 |
| Hash    | `HDEL`                               | delete a key/value                                                                |
| Hash    | `HINCRBY <key> <field> <value>`      | increase a value. auto creates key/field(0)                                       |
| List    | `LLEN`                               | lenght of a set                                                                   |
| List    | `LRANGE`                             | elements of the list (O(S+N); S=start, N=number of elements in range)             |
| List    | `LPUSH <key> <value> [ <value>... ]` | insert values at the head of the list. create the list if not existent.           |
| Set     | `SADD`                               | add to set                                                                        |
| Set     | `SMEMBERS`                           | members of the set (O(N); N=cardinality)                                          |

