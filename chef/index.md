# Chef-Solo


## Terminology
- Node: The deployment target - this is what configured, has chef-client or chef-solo installed
- Chef Client: a command line tool that configures servers.
- Chef Server: This is where all your code resides. It also contains all the information about nodes.
N.B = combined with a Node when using Chef-Solo.
- Recipes - a single file of Ruby code that contains commands to run on a node.
- Resources - a node’s resources include files, directories, users and services.
- Cookbook - a collection of Chef recipes.
- Role: reusable configuration for multiple nodes.
- Attribute: variables that are passed through Chef and used in recipes and templates.
- Template: a file with placeholders for attributes, used to create configuration files.
- Chef-Solo: open source version on Chef Client, doesn’t require access to Chef Server,
requires that a cookbook be on the same physical disk as the node.
- Knife-Solo: provides interface between local Chef-Repo and Chef Server.
- Berkshelf: manages cookbooks and their dependencies (like bundler for rubygems).
- Kitchen: chef-repo, located on workstation.
- WorkStation: local machine aka your laptop. This is where you write your code which 
is then pushed to your chef server. N.B = push to a Node with Chef-Solo.



## Initialize the Project
First create a folder, which will contain all our Chef kitchen:
```sh
mkdir example
cd example
```
Use a bundler to install some gems or install it via shell
```sh
gem install knife-solo
gem install librarian-chef
```
Create a new Kitchen in current directory:
```sh
knife solo init .
```
This will create a file structure like this:
- `.chef/` : stores .pem files and knife.rb
- `cookbooks/` : vendor cookbooks installed with librarian
- `data_bags/` : data bags
- `environments/` : environments
- `nodes/` : nodes
- `roles/` : roles
- `site-cookbooks/` : custom cookbooks

Some useful recap about the used gems: 

### Knife
Provides interface between local Chef Repo and Chef Server.
Knife-Solo adds 5 subcommands to the classic Knife:
- `knife solo init` (create a new kitchen)
- `knife solo prepare` (install chef-solo on target host)
- `knife solo cook` (upload current kitchen to target host and run chef-solo there)
- `knife solo bootstrap` (prepare + cook)
- `knife solo clean` (remove uploaded kitchen from target host)

### Librarian (or BerkShelf)
They manages cookbooks and their dependencies (like bundler for rubygems).
Librarian-Chef can resolve and fetch third-party, publicly-released cookbooks, 
and install them into your infrastructure repository. It can also source 
cookbooks directly from their own source control repositories.
Librarian-Chef takes over your `cookbooks/` directory and manages it for you 
based on your Cheffile. Your Cheffile becomes the authoritative source for the 
cookbooks your infrastructure repository depends on. You should not modify the 
contents of your `cookbooks/` directory when using Librarian-Chef. If you have
cookbooks which are, rather than being separate projects, inherently part of 
your infrastructure repository, then they should go in a separate directory, 
like your `site-cookbooks/` directory, and you do not need to use Librarian-Chef
to manage them.
Every infrastructure repository that uses Librarian-Chef will have a file named 
Cheffile in the root directory of that repository. 

### Kitchen
It is a Chef Repo, is located on a workstation and uploaded to a node when using
Chef-Solo (or to a Chef Server)


## Resource
A Chef resource describes one part of the system, such as a file,
a template, or a package. A Chef recipe is a file that groups related
resources, such as everything needed to configure a web server, 
database server, or a load balancer.
There are 2 types of resources:
1. Platform resource: are avaible in Chef Client directly, doesn't require 
a cookbook
2. Custom resource: must be defined in a resource file located in the `resources/`
folder; used in a Recipe in the same way as a platform resource


Some examples [here](../examples/ex_resource.md)

### Platform Resource
Syntax:
```
TYPE 'NAME' do
  PROPERTY_NAME 'PROPERTY_VALUE'
  action :TYPE_OF_ACTION
end
```
- type: platform resource types: package, template, service,... 
- name: for platform resources dealing with files 
(directory, cookbook_file, etc.) this is usually the path of 
corresponding file on chef node.

Popular platform resources:  

| Resource | Desc   |
| -------- | ------ |
|  script  | execute script using specified interpretor |
|   bash   | execute script using Bash interpretor      |
|   ruby   | execute script using Ruby interpretor      |
|   link   |  	create sym or hard links |
| directory | manage directories   |
| cookbook_file | transfer files from subdirectory of `files/`     |
| template | transfer files from subdirectory of `templates/`    |
|  package | install package (rpm, deb, etc.)     |
| gem_package | install gem system-wide |
|  chef_gem | install gem into the instance of Ruby dedicated to chef-client    |
|  cron | modify cron entries    |
|  user | manage users   |  




### Custom Resource
A brief introduction to the custom resource.
It declares properties of custom resource, loads current properties if resource 
already exists,defines all resource actions,

Syntax:
```
resource_name :httpd

property :instance_name, String, name_property: true
property :name, RubyType, default: 'value'

default_action :action_2

load_current_value do
  # some Ruby
end

action :action_1 do
 # a mix of built-in Chef resources and Ruby
end

action :action_2 do
 # a mix of built-in Chef resources and Ruby
end
```
- resource_name overrides default resource name
- name_property: true makes this property to use resource name as its value
- load_current_value block loads current values for all properties
- default_action overrides default action



## Recipes
A Chef cookbook is comprised of recipes that a nodes desired state. 
Recipes are written in Ruby and contain information about everything that needs 
to be run, changed, or created on a node. Recipes work as a collection of 
resources that determine the configuration or policy of a node, with resources
being a configuration element of the recipe. For a node to run a recipe, it 
must be on that node’s run list.

The dafault recipe (is necessary in every cookbook) is `default.rb`.

Recipe can include other recipes from other cookbooks using include_recipe method:
```
include_recipe 'apache2::mod_ssl'
```

Included recipe must be declared as dependency in metadata.rb:
```
depends 'apache2'
```

[Here](../examples/ex_recipe) you can find an axemple of a recipe that is part of Chef’s Vim cookbook. 
It is in charge of installing the required Vim package based on a node’s Linux distribution.


## Cookbooks
A cookbook is a collection of Chef Recipes. All cookbooks like a Chef are written in Ruby. 

[Here](../examples/ex_cookbook.md) you can find an example about the structure of a cookbook.

### Vendor Cookbooks

#### Using Librarian
Create a Cheffile in the directory of the project:
```sh
librarian-chef init
```
Add dependencies and their sources to the Cheffile:
```
site 'https://supermarket.getchef.com/api/v1'
    cookbook 'ntp'
    cookbook 'timezone', '0.0.1'
    cookbook 'rvm',
      :git => 'https://github.com/fnichol/chef-rvm',
      :ref => 'v0.7.1'
    cookbook 'cloudera',
      :path => 'vendor/cookbooks/cloudera-cookbook'
```
Then run:
```
librarian-chef install
```
This command looks at each cookbook declaration and fetches the cookbook 
from the source specified, or from the default source if none is provided.
Each cookbook is inspected, its dependencies are determined, and each dependency
is also fetched. For example, if you declare cookbook 'nagios', which depends 
on other cookbooks such as 'php', then those other cookbooks including 'php' 
will be fetched. This goes all the way down the chain of dependencies.
This command then copies all of the fetched cookbooks into your `cookbooks/`
directory, overwriting whatever was there before.

#### Using BerkShelf
In the directory of the project create a `Berksfile` and inside it write:
```
cookbook 'apache2'
```
Then run this to install cookbook globally into `~/.berkshelf/cookbooks/`:
```sh
berks install
berks upload
```
Or run this to install cookbook both locally into `cookbooks/` and globally into 
`~/.bershelf/cookbooks/`:
```sh
berks vendor cookbooks
```


### Site Cookbooks
Let’s create our cookbook. Our custom cookbooks should be 
in the folder `site-cookbooks/` (the folder `cookbooks/` is used for vendor 
cookbooks and managed by librarian or berkshelf)
Create a custom Cookbook:
```sh
cd site-cookbooks/
knife cookbook create Tomatoes
```
Then configure and add your service using the Recipes,Attributes,DataBags ecc.

Lastly add the new recipe in the `run_list` of the project Node.



## Nodes
Nodes are configured in node files for chef-solo only;
chef-client interacts with chef server to retrieve node configuration.
Consequently, node-specific attributes must be located in a 
JSON file on the target system, a remote location, or a web server on the local network.
- Nodes contains node-specific attributes and run-list
- Nodes represents machine (physical, virtual, etc.)
- Nodes separate JSON file for each node in `nodes/`

Node file name (as a rule): DOMAIN.json
Create a `DOMAIN.json` in the `nodes` folder and write inside it:
```
{
  run_list: [
        'role[base]',
        'recipe[apache2]'
  ]
}
```

N.B = `run_list` is the main key in a node file, the `default.rb` is executed when 
only cookbook name is specified without specific recipe.
Run lists define which recipes a node will use. The run list is an ordered list 
of all roles and recipes that the chef-client needs to pull from the Chef server 
to run on a node. Roles are used to define patterns and attributes across nodes.
N.B = We can use the `run_list` of a specific role declaring it in the `run_list` of the node.



## Roles 
In most cases you project contain several servers with the same configuration.
For example, you can have several web servers and one load balancer, which 
balance on this web servers. Or you can have several database or queue servers
with identical configuration. In this case, it is very hard way to clone each 
server by nodes, because you need copy all attributes from one node to another.
Maintain a system with such nodes also will be hard: you will have to modify 
the “n” number of nodes to change some attribute value. We can use role! 
A role provides a means of grouping similar features of similar nodes, 
providing a mechanism for easily composing sets of functionality.
A Role is a server template (web, database, etc.),
it contains server-specific run-list and attributes.
Need to create separate JSON file for each role in `roles/`.
The json file required:
- name : unique name (usually the same as file name)
- description
- chef_type : role
- json_class : Chef::Role
- run_list : role-specific run-list (may include other roles).

An example of a role file can be found [Here](../examples/ex_role.json)



## Data Bags
Data Bag are global variable stored as JSON data;
it is used to store sensitive data (credentials, etc.).
data bag consists of data bag items: DATA_BAG/DATA_BAG_ITEM.json

The structure is like this:
- name : unique name (usually the same as file name)
- chef_type : data_bag_item
- json_class : Chef::DataBagItem
- data_bag :  	data bag name
- raw_data : attributes of data bag item (id attribute is required)

Example [here](../examples/ex_databag.json)


## Examples

### Example Cookbook
## Structure
A cookbook can have:

- metadata.rb - a file, which contain all information about the cookbook (name, dependencies).
```
name              "nginx"
maintainer        "Opscode, Inc."
maintainer_email  "cookbooks@opscode.com"
license           "Apache 2.0"
description       "Installs and configures nginx"
version           "1.1.2"

recipe "nginx", "Installs nginx package and sets up configuration with Debian apache style with sites-enabled/sites-available"
recipe "nginx::source", "Installs nginx from source and sets up configuration with Debian apache style with sites-enabled/sites-available"

%w{ ubuntu debian centos redhat amazon scientific oracle fedora }.each do |os|
 supports os
end

%w{ build-essential }.each do |cb|
 depends cb
end

depends 'ohai', '>= 1.1.2'

%w{ runit bluepill yum }.each do |cb|
 recommends cb
end
```

----------------------------------------------------------------------------------------------

- attributes - Attributes define specific values about a node and its configuration
and are used by the Chef client to apply those attributes to nodes via its 
attribute list. The chef client can receive attributes from nodes, attribute files,
recipes, environments, and roles. Often attributes are used in conjunction with 
templates and recipes to define settings. :
```
default['nginx']['version'] = "1.2.3"
default['nginx']['dir'] = "/etc/nginx"
default['nginx']['log_dir'] = "/var/log/nginx"
default['nginx']['binary'] = "/usr/sbin/nginx"
```

----------------------------------------------------------------------------------------------

- files - These are static files that can be uploaded to nodes. Files can be 
configuration and set-up files, scripts, website files. For example, you may 
have a recipe that uses an index.php file. You can use a cookbook_file resource 
block within a recipe to create the file on a node. All static files should be 
stored in a cookbook’s files directory

----------------------------------------------------------------------------------------------

- recipes - a folder, which contain all recipes from this cookbook. 
Each recipe is in a separate Ruby file

----------------------------------------------------------------------------------------------

- templates - are embedded Ruby files (.erb) that are used to dynamically
create static text files. To use a template within a cookbook, you must
declare a template resource in a recipe and include a corresponding .erb 
template file in a template subdirectory. Your template resource can contain
variables that will then be used by the template to dynamically provide those
values based on a nodes particular context.

----------------------------------------------------------------------------------------------


### Example Databag
```
{
  "name":"data_bag_item_dogs_tibetanspaniel",
  "json_class":"Chef::DataBagItem",
  "chef_type":"data_bag_item",
  "data_bag":"dogs",
  "raw_data":
    {
      "description":"small dog that likes to sit in windows",
      "id":"tibetanspaniel"
    }
}
```

### Example Recipe
```
vim_base_pkgs = value_for_platform_family(
  %w(debian arch) => ['vim'],
  %w(rhel fedora) => ['vim-minimal', 'vim-enhanced'],
  'default' => ['vim']
)

package vim_base_pkgs

package node['vim']['extra_packages'] unless node['vim']['extra_packages'].empty?
```

### Example Resource
#### Platform Resource

Create a file in the `tmp/` folder named `hi` and set its content:
```
file '/tmp/hi' do
  content 'hi world'
end
```

Delete a file named as `hi` in the `tmp/` folder:
```
file '/tmp/hi' do
  action :delete
end
```

Create a directory named as `something` and set some permissions:
```
directory '/tmp/something' do
  owner 'root'
  group 'root'
  mode 00755
  action :create
end
```

This is a Recipe, it is shown to highlight the differences 
in types of resources (package, services, built in).
It consists of installing and setting up different services.
N.B = calling a `package` will install it
```
apt_update 'Update the apt cache daily' do
  frequency 86_400
  action :periodic
end

package 'apache2'

service 'apache2' do
  supports status: true
  action [:enable, :start]
end

file '/var/www/html/index.html' do
  content '<html>
  <body>
    <h1>hello world</h1>
  </body>
</html>'
end
```

#### Custom Resource
Homepage is a property that sets the default HTML 
for the index.html file with a default value.
The action block uses the built-in collection of resources to 
tell Chef Infra Client how to install Apache, start the service, 
and then create the contents of the file located at /var/www/html/index.html . 
action :create is the default resource, because it is listed first; 
action :delete must be called specifically (because it is not the default resource)
```
property :homepage, String, default: '<h1>Hello world!</h1>'

action :create do
  package 'httpd'

  service 'httpd' do
    action [:enable, :start]
  end

  file '/var/www/html/index.html' do
    content new_resource.homepage
  end
end

action :delete do
  package 'httpd' do
    action :delete
  end
end
```

### Example Role
```
{
  "name": "base",
  "chef_type": "role",
  "json_class": "Chef::Role",
  "description": "The base role for systems",
  "run_list": [
    "recipe[apt]",
    "recipe[sudo]",
    "recipe[sysctl]",
    "recipe[nginx::source]",
    "recipe[Tomatoes]"
  ]
}
```

