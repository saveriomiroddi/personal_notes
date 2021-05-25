# Bundler

- [Bundler](#bundler)
  - [Generic structure](#generic-structure)
  - [Gem definition options](#gem-definition-options)

## Generic structure

```ruby
source 'https://rubygems.org'

# If there is a gemspec: add here `gemspec`, with rake/rspec as dev dependencies.
#
gemspec

# If there isn't a gemspec: add here the `development` group, with rspec.
# The rake gem can be excluded unless specifically required.
#
group :development do
  # gem "rake"
  gem 'rspec', '~> 3.10.0'
end

# The test group should include gems that are not strictly required.
#
group :test do
  gem "coveralls_reborn", "~> 0.22.0"
  gem "simplecov", "~> 0.21.2"
end

# Multi-group gem.
# This specific case is a Rails default addition.
#
group :development, :test do
  gem 'byebug'
end
```

## Gem definition options

```ruby
# Local repository.
#
gem 'sshkit-sudo', '~> 0.2.0', path: '/path/to/sshkit-sudo'

# Git repository; `branch` is optional; `ref` and `tag` are also supported.
#
gem 'sshkit-sudo', git: 'https://github.com/ticketsolve/sshkit-sudo', branch: 'mytest'
```
