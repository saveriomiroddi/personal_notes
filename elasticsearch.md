# Elasticsearch

- [Elasticsearch](#elasticsearch)
  - [Backup](#backup)

## Backup

```sh
# Register a backup location, of type "filesystem" ("fs")
# Directory is created if it doesn't exist.
#
# On a test, compress and no compress gave the same end size, with the second option being
# much faster, and the xz compression of both leading to equal result.
#
curl -XPUT 'http://localhost:9200/_snapshot/my_backup' -d '{
  "type": "fs",
  "settings": {
    "location": "/tmp/elasticsearch_dump",
    "compress": false
  }
}'

# Backup!
#
curl -XPUT "localhost:9200/_snapshot/my_backup/backup_140630?wait_for_completion=true"

# Restore!
#
curl -XPOST "localhost:9200/_snapshot/my_backup/backup_140630/_restore?wait_for_completion=true"

# Delete the backup location.
#
curl -XDELETE 'http://localhost:9200/_snapshot/my_backup'
```
