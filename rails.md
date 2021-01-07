# Rails

- [Rails](#rails)
  - [Controllers](#controllers)
  - [ActiveRecord](#activerecord)
  - [Migrations](#migrations)

## Controllers

Predefined variables:

- `action_name`: current action name

## ActiveRecord

Query hints (6.0+):

```ruby
Article.joins(:user).optimizer_hints("JOIN_ORDER(articles, users)").to_sql
# => SELECT /*+ JOIN_ORDER(articles, users) */ `articles`.* FROM `articles` INNER JOIN `users` ON `users`.`id` = `articles`.`user_id`
```

## Migrations

Reference: https://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements

Table creation/table column operations:

```ruby
create_table id: false, primary_key: :column, options: 'ENGINE=MyISAM' do |t|
  # :type: can be a string, e.g. 'MEDIUMINT UNSIGNED NOT NULL'
  #
  # :column_options (valid for all the column methods):
  #   :null:    true/false
  #   :limit:   int; number of chars for string, bytes otherwise
  #   :default
  #
  t.column :column_name, :type, **column_options

  t.string
  t.text :info, limit: 64.kilobytes, null: false    # create a mediumtext (<64k is text)
  t.integer
  t.float
  t.decimal
  t.datetime
  t.timestamp
  t.time
  t.date
  t.binary
  t.boolean
  t.timestamps

  t.index :column_name{, column_name_N}, unique: true, name: index_name
end

# The create_table methods also apply here.
# :bulk perfoms all the changes in one statement (defaults to false)
#
change_table :table, bulk: true do |t|
  t.change                             # changes the column definition
  t.change_default                     # changes the column default
  t.rename                             # rename a column
  t.references
  t.belongs_to

  t.remove :column_name                # remove a column
  t.remove_index :column_name{, column_name_N}, name: index_name
  t.remove_references
  t.remove_belongs_to
  t.remove_timestamps
end
```

Direct operations:

```ruby
rename_table :old_table_name, :new_table_name
drop_table :table_name
change_column :table, :column, :type, **column_options # see options above
change_column_default :table, :column, :default
```

Irreversible migration error:

```ruby
raise ActiveRecord::IrreversibleMigration
```
