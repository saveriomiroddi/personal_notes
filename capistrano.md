# Capistrano

- [Capistrano](#capistrano)
  - [Actions](#actions)

## Actions

In order to rewrite a task, the previous one must be cleared:

```ruby
Rake::Task['qualified_task_name'].clear_actions
```

If other tasks' callbacks invoke the it:

- if it's redefined, it will be invoked;
- otherwise, it's not invoked, and no error is raised.
