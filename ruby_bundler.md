# Bundler

- [Bundler](#bundler)
  - [Gem definition options](#gem-definition-options)

## Gem definition options

```ruby
# Local repository.
#
gem 'sshkit-sudo', '~> 0.2.0', path: '/path/to/sshkit-sudo'

# Git repository; `branch` is optional; `ref` and `tag` are also supported.
#
gem 'sshkit-sudo', git: 'https://github.com/ticketsolve/sshkit-sudo', branch: 'mytest'
```