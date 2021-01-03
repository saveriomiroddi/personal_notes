# Rails

- [Rails](#rails)
  - [Controllers](#controllers)
  - [ActiveRecord](#activerecord)
  - [Migrations](#migrations)

## Controllers

```ruby
action_name           # current action name
```

## ActiveRecord

Query hints (6.0+):

```ruby
Article.joins(:user).where(true).optimizer_hints("JOIN_ORDER(articles, users)").to_sql
# => SELECT /*+ JOIN_ORDER(articles, users) */ `articles`.* FROM `articles` INNER JOIN `users` ON `users`.`id` = `articles`.`user_id` WHERE (TRUE)
```

## Migrations

Operations:

```ruby
rename_table :old_table_name, :new_table_name
drop_table :table_name
```