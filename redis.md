# Redis

## Commands

| Type    |               Command                | Comment                                                                           |
| :------ | :----------------------------------: | --------------------------------------------------------------------------------- |
| General |                `del`                 | delete a key                                                                      |
| Hash    |                `hget`                | get a key                                                                         |
| Hash    |              `hgetall`               | get all the fields/values                                                         |
| Hash    |               `hmset`                | set multiple fields/values      (ruby library: use mapped_hmset for hash sources) |
| Hash    |                `hset`                | set a field/value                                                                 |
| Hash    |    `hsetnx <key> <field> <value>`    | set a field/value, if the field doesn't exist. auto creates keys.                 |
| Hash    |                `hdel`                | delete a key/value                                                                |
| Hash    |   `hincrby <key> <field> <value>`    | increase a value. auto creates key/field(0)                                       |
| List    |                `llen`                | lenght of a set                                                                   |
| List    |               `lrange`               | elements of the list (O(S+N); S=start, N=number of elements in range)             |
| List    | `lpush <key> <value> [ <value>... ]` | insert values at the head of the list. create the list if not existent.           |
| Set     |                `sadd`                | add to set                                                                        |
| Set     |              `smembers`              | members of the set (O(N); N=cardinality)                                          |

