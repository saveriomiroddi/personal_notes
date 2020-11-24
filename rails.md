# Rails

- [Rails](#rails)
  - [Controllers](#controllers)
  - [ActiveRecord](#activerecord)

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