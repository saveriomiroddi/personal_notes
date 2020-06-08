# Chef

- [Chef](#chef)
  - [Resource concepts](#resource-concepts)
    - [Attributes precedence/merging](#attributes-precedencemerging)
    - [Basic concepts](#basic-concepts)
    - [Subscribe/notify](#subscribenotify)
    - [Available variables](#available-variables)
  - [Resources](#resources)
    - [`bash`](#bash)
    - [`directory`](#directory)
    - [`execute`](#execute)
    - [`remote_file`](#remote_file)
  - [Tools](#tools)
    - [Knife](#knife)

## Resource concepts

### Attributes precedence/merging

Lower to higher:

- A default attribute located in a cookbook attribute file
- A default attribute located in a recipe
- A default attribute located in an environment
- A default attribute located in role
- A force_default attribute located in a cookbook attribute file
- A force_default attribute located in a recipe
- A normal attribute located in a cookbook attribute file
- A normal attribute located in a recipe
- An override attribute located in a cookbook attribute file
- An override attribute located in a recipe
- An override attribute located in a role
- An override attribute located in an environment
- A force_override attribute located in a cookbook attribute file
- A force_override attribute located in a recipe
- An automatic attribute identified by Ohai at the start of the chef-client run

Merging:

- An environment default Array attribute will merge the values with a recipe default one
- An environment overrider Array attribute will replace the values of a recipe default one

### Basic concepts

```ruby
# Lazily execute a command
lazy { <commands> }

# Conditional resource execution
only_if { ::File.exist?(extract_path) }
not_if { ::File.exist?(extract_path) }

# Always use both; if `group` is omitted, it will default to `root`.
user  "application"
group "application"
```

### Subscribe/notify

```ruby
file '/etc/nginx/ssl/example.crt' do
  mode  '0600'
  owner 'root'
end

# Don't forget that the :subscribes action is the action of the enclosing resource!
service 'nginx' do
  action :nothing
  subscribes :reload, 'file[/etc/nginx/ssl/example.crt]', :immediately
end
```

### Available variables

```ruby
Chef::Config[:cookbook_path]      # Cookbook path, usable in recipes
```

## Resources

### `bash`

```ruby
bash 'extract_module' do
  cwd    ::File.dirname(src_filepath)
  code   "code"
  action :run
end
```

### `directory`

Watch out! One can't use two resources with the same name, for creation and (cleanup-time) deletion, as they will at least cause problems with notifications.

```ruby
directory '/etc/apache2' do
  mode      '0755'
  recursive false         # If true and :create, owner/group apply only to the leaf!
  action    :create       # :delete (don't forget :recursive=true!)
end
```

### `execute`

Execute a command.
Each command must be in an execute block, otherwise they will be all executed even if one of them fails

```ruby
execute "Description" do
  command "command"
end
```

### `remote_file`

```ruby
remote_file local_tarball_filename do
  source node[:mysql_server][:tarball_uri]
end
```

## Tools

### Knife

Execute code on nodes from the commandline (including recipes doesn't work):

```sh
knife exec -E <<~RUBY
  nodes.transform(:all) do | n |
    if n.hostname == 'vmsav2'
      puts n.normal_attrs.keys
    end
  end
RUBY
```
