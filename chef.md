# Chef

- [Chef](#chef)
  - [Resource concepts](#resource-concepts)
    - [Attributes precedence/merging](#attributes-precedencemerging)
    - [Basic concepts](#basic-concepts)
    - [Subscribe/notify](#subscribenotify)
    - [Available variables](#available-variables)
  - [Resources](#resources)
    - [`bash`](#bash)
    - [`execute`](#execute)
    - [`cron`](#cron)
    - [`cookbook_file`](#cookbook_file)
    - [`directory`](#directory)
    - [`execute`](#execute-1)
    - [`line` (manual file editing)](#line-manual-file-editing)
    - [`mount`](#mount)
    - [`package`/`dpkg_package`](#packagedpkg_package)
    - [`remote_file`](#remote_file)
    - [`systemd_unit`](#systemd_unit)
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

# Notification happens when the referenced resource state is changed.
# Don't forget that the :subscribes action is the action of the enclosing resource!
#
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

The attributes, are the ones specified in the example resource, unless specified.

### `bash`

```ruby
bash 'extract_module' do
  cwd    ::File.dirname(src_filepath)
  code   "code"
  action :run
end
```

### `execute`

```ruby
execute 'apache_configtest' do
  command '/usr/sbin/apachectl configtest'
  action  :run
end
```

### `cron`

```ruby
cron 'job_name' do
  command          String

  minute           Integer, String         # default value: "*"
  hour             Integer, String         # default value: "*"
  day              Integer, String         # default value: "*"
  month            Integer, String         # default value: "*"
  weekday          Integer, String, Symbol # default value: "*"

  user             String                  # default value: "root"

  environment      Hash
  home             String
  mailto           String
  path             String
  shell            String
  time             Symbol
  time_out         Hash
end
```

### `cookbook_file`

```ruby
cookbook_file '/etc/systemd/system.conf' do
  source    'etc/systemd/system.conf'
  mode      '0644'        # best to always specify; the default (not in all case) is 0777 + umask -> typically 0755.
end
```

### `directory`

Watch out! One can't use two resources with the same name, for creation and (cleanup-time) deletion, as they will at least cause problems with notifications.

```ruby
directory '/etc/apache2' do
  mode      '0755'        # default = 0777 + umask -> typically 0755; if recursive, the permission are applied to the entire created tree
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

### `line` (manual file editing)

If manual editing is required, **do not use Chef::Util::FileEdit** as widely suggested (see https://git.io/JURBC), instead, use this cookbook.

Examples (see [repo](https://supermarket.chef.io/cookbooks/line)):

```ruby
# Append a line, if it doesn't exist; matches exactly.
#
append_if_no_line "title" do
  path "/path/to/file"
  line "exact_line_content"
end

# Replaces a line, or appends it; matches a pattern.
#
replace_or_add "title" do
  path              "/path/to/file"
  pattern           "regex.*"
  line              "replacement"

  remove_duplicates false
  replace_only      false            # don't append, if there are no matches
end
```

### `mount`

```ruby
# Generates:
#
# "fs-id:/dir /path/to/mountdir efs defaults,tls,accesspoint=fsap-id 0 0"
#
mount "/path/to/mountdir" do
  device  "fs-id:/dir"
  fstype  "efs"                               # default: `auto`
  options "defaults,tls,accesspoint=fsap-id"
  pass    0                                   # default: 2

  action [:enable, :mount]
end
```

### `package`/`dpkg_package`

There is currently no way of installing a local deb package, and pull the dependencies (the `gdebi` cookbook does it, but it's unmaintained).

```ruby
package 'package_name' do
  action :install                       # :purge, ...
  version '1.3.4-2'                     # optional
end

# Version with multiple packages; it installs them faster, and has a more compact output.
# The `version` attributes takes an Array, in this case.
#
package %w(package1 package2)
```

```ruby
dpkg_package 'package_name' do
  source '/path/to/local_filename.deb'
end
```

### `remote_file`

```ruby
remote_file local_tarball_filename do
  source node[:mysql_server][:tarball_uri]
end
```

### `systemd_unit`

```ruby
systemd_unit 'nmon.service' do
  content <<~UNIT
    [Unit]
    Description=Nmon system stats recording

    [Service]
    Type=forking

    StandardOutput=syslog
    StandardError=syslog
    SyslogIdentifier=nmon

    ExecStart=$nmon_location -F #{node[:monitoring][:nmon_stats_file]} -s #{node[:monitoring][:nmon_collection_interval]}
    ExecReload=/bin/kill -HUP $MAINPID

    Restart=on-failure

    [Install]
    WantedBy=multi-user.target
  UNIT

  # Systemd can be overly strict when verifying units, so in certain cases it is preferable not to verify the unit.
  #
  verify false

  action [:create, :enable, :reload_or_restart]
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
