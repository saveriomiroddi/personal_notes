# Capistrano

- [Capistrano](#capistrano)
  - [Tasks/Actions](#tasksactions)

## Tasks/Actions

In order to rewrite a task, the previous one must be cleared:

```ruby
Rake::Task['qualified_task_name'].clear_actions
```

If other tasks' callbacks invoke the it:

- if it's redefined, it will be invoked;
- otherwise, it's not invoked, and no error is raised.

Watch out! If the task is invoked by an :after hook, the hook won't be cleared; see [this question](https://stackoverflow.com/q/22712240).
