# Vagrant


## Box

### Download Box
It is possible to download from the [Vagrant Cloud](https://app.vagrantup.com/boxes/search)
some images pre-created by users and the hashicorp. 
From shell just type:
```sh
vagrant box add hashicorp/precise64
```
This will download (if not already present in the image list)
the box precise64 ie an Ubuntu 12.04 64 bit system.


### Box List
To view the list of all the boxes present (downloaded), just type:
```sh
vagrant box list
```

### Delete Box
To delete a Box from a shell just type:
```sh
vagrant delete hashicorp/precis64
```
It is necessary to pass the id of the box to delete the selected box.

### Base Box
Vagrant provides BaseBox, that is a minimal image of an OS.
An example of a BaseBox that can be found on the VagrantCloud is ubuntu/bionic64.
In a BaseBox there are:
- Package manager
- SSH
- SSH user so Vagrant can connect
- Chef, Puppet, etc. but not strictly required.  

For advanced users it can be useful to create customized 
BaseBoxes (that are provider-specific).
Since the topic is very large (memory settings, disk, peripherals, ...) 
all the informations are avaible [here](https://www.vagrantup.com/docs/boxes/base.html).
Furthermore Vagrant has a suite that allows you to automate the creation of BaseBox called [Packer](https://www.packer.io/).
For most users the boxes on the VagrantCloud are enough.



## Initialize Vagrant Box
Each project in Vagrant must be placed in a personal directory,
and to work with it you need to move to the
chosen project. It is necessary to create a new directory, move
inside it and initialize the project
```sh
mkdir project1
cd project1
vagrant init hashicorp/precise64
```
This will create a `Vagrantfile` file containing all the
characteristics of the project, in this case it will be:
```Vagrantfile
Vagrant.configure ("2") do | config |
   config.vm.box = "hashicorp/precise64"
end
```
Now all we have is a VM (not started) based on Ubuntu 12.04 64 bit.

Here are presented some useful flag:
- Create a minimal Vagrantfile (no comments or helpers):
```sh
vagrant init -m hashicorp/precise64
```
- Create a new Vagrantfile, overwriting the one at the current path:
```sh
vagrant init -f hashicorp/precise64
```
- Create a Vagrantfile with the specific box, from the specific box URL:
```sh
vagrant init my-company-box https://boxes.company.com/my-company.box
```
- Create a Vagrantfile, locking the box to a version constraint:
```sh
vagrant init --box-version '> 0.1.5' hashicorp/precise64
```


## Management Vagrant Project
Vagrant provides several commands to interact with the VMs:
- Stop the VM:
```sh
vagrant halt
```
if it is used in a folder containing several VMs are all 
stopped unless unless the id of the VM to be stopped is passed.

- Suspend the VM:
```sh
vagrant suspend
```
if it is used in a folder containing several
VMs are all suspended unless the id of the VM to be suspended is passed.

- Delete the VM:
```sh
vagrant destroy
```
this command is a bit different from the previous ones, in fact if executed
inside a folder containing a project with multiple VMs, from there
possibility to choose in an interactive way which to delete and
which not; instead if the id is specified it is possible to delete the
VM wherever it is.

- Interact with the VM:
After the VM has been activated, an SSH session can be started
moving within the project directory and declaring
the hostname of the VM to use
```sh
cd project1
vagrant ssh
```
From now on you will have control over the VM, to close the
session just press CTRL + D or
```sh
logout
```

- Shared folders
It is possible to share resources between the host machine and the
guest machine (VM) if they are contained within
of the folder containing the VagrantFile, to verify it
once the SSH session with the VM is open, just type
```sh
ls /vagrant
```
All that is present in the folder is visible either
from the host to the guest.



## Start Vagrant Project
### First Start
If the project has just been initialized to start the VM you need to
move inside the project folder and start it:
```sh
cd project1
vagrant up
```
In this way provisioning is forced (performed through Shell,
Chef, Puppet, ...) and the guest system is started.

### Generic Start
If the project has already been started previously is recommend for
start it using the command:
```sh
cd project1
vagrant reload
```
This command allows the machine to start without them coming
perform the reloading operations on the provisioning, if they have been
carried out provisioning operations to make them effective is
necessary to launch:
```sh
cd project1
vagrant reload --provision
```


## Status Vagrant Project
To view the status of the VMs there are two commands to execute
in different positions:
- vagrant status (local)
- vagrant global status (global)

The status are three:
- `not created`
- `running`
- `stopped`

### Vagrant Status

This command returns the status of the vm:
```sh
cd project1
vagrant status
```
It will return the status of the machines based on where it is executed (eg project1):

### Vagrant Global Status

This command allows us to view the status of all the VMs previously created.
Often the status is not updated so is recommended to force the reload of the status:
```sh
vagrant global-status --prune
```




## Networking Configurations
Each of the networking adapters can be separately configured
to operate in one of the following modes:
- NAT: abbreviation of Network Address Translation,
a VM with NAT acts like a real computer that
connects to the Internet through a router, in this
case the Virtualbox Networking Engine. Used for example,
for browse Web,download file, view email,... inside the guest.
- NAT Network: Virtualbox behaves like a DHCP server by 
assigning an IP to the guest but from a different address space.
Shortly is a type of internal network that allow outbounds connections.  
N.B = create a NAT Network first, in VirtualBox click
File->Preferences->Network->NAT Networks, assign a name
a CIDR and entering support for DHCP.
- Bridged: with this configuration Virtualbox uses a network card on
the host system and exchange network packets directly avoidind the
host operating system's network stack.  
N.B = In the VM Network settings you need to specify which network card
you want to use.
- Internal: is protected network that is not accessible to the internet
in general. 
- Host Only: VMs in this network can connect to each other and the same
would be reachable via your host machine but the VMs would not be able 
to access the outside external network.
- Generic: rarely used modes which share the same generic network 
interface, by allowing the user to select a driver which can be included
with Oracle VM VirtualBox or be distributed in an extension pack. 

|                          Question                          | NAT | NAT Network | Bridged | Internal | Host Only |  
| ---------------------------------------------------------- | --- | ----------- | ------- | -------- | --------- |
|                  Can VM connect to Host?                   | Yes |     Yes     |   Yes   |    No    |    Yes    |
|                  Can Host connect to Vm?                   | PF  |     PF      |   Yes   |    No    |    Yes    |
|           Can VM connect to external network?              | Yes |     Yes     |   Yes   |    No    |    No     |
|       Can VM connect another VMS in the same network?      | No  |     Yes     |   Yes   |    Yes   |    Yes    |
| Can other computers on the external network connect to VM? | PF  |     PF      |   Yes   |    No    |    No     |

**PF means that is not default but can achieve that doing a port forward**

[More info from here](https://www.virtualbox.org/manual/ch06.html#natforward)



## Vagrantfiles
VagrantFile is the heart of Vagrant, it allows us to define
all project specifications. For each project it is allowed
just a Vagrant file.  
N.B = if you want to use the Vagrant files commented below
it is necessary to rename the files in Vagrantfile

### [Base Vagrantfile](../vagrantfiles/Vagrantfile_base)
This Vagrantfile is the standard one created at the
`init` of the project.
This Vagrantfile create a single VM with a specific box.


### Networking Settings
In Vagrant the adapter n.1 (eth0) is always a NAT adapter
because is used from the Host to communicate with the Guest via SSH.

#### Private Network
Vagrant private networks allow you to access your guest machine 
by some address that is not publicly accessible from the global Internet.

- [Host Only](../vagrantfiles/Vagrantfile_privatenetwork_host_only):
By default, private networks are host-only networks, 
because those are the easiest to work with.
Behind the scene, VirtualBox creates a new virtual interface ("loopback")
on the host which appears next to the existing network interfaces.
VirtualBox even provide a built-in DHCP server for host-only networking
if no static IPs have been assigned.

- [Internal Network](../vagrantfiles/Vagrantfile_privatenetwork_internal):
The Vagrant VirtualBox provider supports using the private network as
a VirtualBox internal network. By default, private networks are host-only
networks, because those are the easiest to work with. However, 
internal networks can be enabled as well.

#### Public Network
Vagrant public networks are less private than private networks, 
and the exact meaning actually varies from provider to provider,
hence the ambiguous definition. The idea is that while private 
networks should never allow the general public access to your
machine, public networks can.
It's a Bridged Network.
- [Bridged](../vagrantfiles/Vagrantfile_publicnetwork)


#### [Forwarded Ports](../vagrantfiles/Vagrantfile_forwardedport) 
Vagrant forwarded ports allow you to access a port on your host 
machine and have all data forwarded to a port on the guest machine,
over either TCP or UDP.
For example: If the guest machine is running a web server listening 
on port 80, you can make a forwarded port mapping to port 8080 
(or anything) on your host machine. You can then open your browser 
to localhost:8080 and browse the website, while all actual network
data is being sent to the guest.

### Provider Settings
While well-behaved Vagrant providers should work with any Vagrantfile 
with sane defaults, providers generally expose unique configuration 
options so that you can get the most out of each provider.
This provider-specific configuration is done within the Vagrantfile 
in a way that is portable, easy to use, and easy to understand.

- [VirtualBox](../vagrantfiles/Vagrantfile_provider):
  VBoxManage is a utility that can be used to make modifications to 
  VirtualBox virtual machines from the command line.
  Vagrant exposes a way to call any command against VBoxManage
  just prior to booting the machine.
  [Here](https://www.virtualbox.org/manual/ch08.html) several commands
  are presented.


### [Multi Machine](../vagrantfiles/Vagrantfile_multi_machine)
Vagrant is able to define and control multiple guest machines per
Vagrantfile. This is known as a "multi-machine" environment.
These machines are generally able to work together or are somehow 
associated with each other. 

- [Multi with same Provider](../vagrantfiles/Vagrantfile_multi_sameprovider)
  This will create a lab with 3 machine, `ubonda` has access to internet 
  with a bridged interface; the other two are in a private network
  and can communicate only with each other but not with `ubonda`.
  It is necessary to declare a `virtualbox__intnet: true` to isolate the two
  machine or `ubonda` will connect to them.
  
- [Multi with different Provider](../vagrantfiles/Vagrantfile_multiprovider_subnet)
  In this lab I will use 3 machines:
  1. `ubonda` a vm created with Virtualbox that can navigate through internet and 
  connect to `slave-connector`
  2. `slave-connector` a container created with Docker that is in a subnet with
  another container `slave`and can communicate with `ubonda`
  3. `slave` a container created with Docker that is in a subnet with `slave-connector`
  First of all create a docker network to connect `ubonda` with `slave-connector`:
  ```sh
  docker network create bridge2 --gateway=192.168.50.1 --subnet=192.168.50.1/24
  ```
  Then check the network id just created:
  ```sh
  docker network ls
  ```
  Finally put the id in the `public network` in the Vagrantfile.
  Remember to use the `docker_network__internal: true` to create a private
  network for the containers.
  
### Provisioning
Provisioners in Vagrant allow you to automatically install software,
alter configurations, and more on the machine as part of the vagrant up
process.  
This is useful since boxes typically are not built perfectly for your use
case. Of course, if you want to just use vagrant ssh and install the 
software by hand, that works. But by using the provisioning systems 
built-in to Vagrant, it automates the process so that it is repeatable.

- [Shell Provisioning](../vagrantfiles/Vagrantfile_shellprovision):
The Vagrant Shell provisioner allows you to upload and execute a 
script within the guest machine.
Shell provisioning is ideal for users new to Vagrant who want 
to get up and running quickly.

- [Puppet Provisioning](../vagrantfiles/Vagrantfile_puppetprovision):
N.B = puppet need to be installed in the guest vm.  
By default, Vagrant will configure Puppet to look for manifests in the "manifests"
folder relative to the project root, and will use the "default.pp" manifest as an entry-point.
In my case i use [ubonda.pp](../vagrantfiles/ubonda.pp) in this file you have to insert 
the modules present in the modules folder and that you want to install.
Every modules need to contain an [init.pp](../vagrantfiles/init.pp), a file containing all
the services, packages and commands to be executed, the syntax is in Puppet Language.



## Questions
- **It is possible to use two or more provider on a single Vagrantfile
different providers?**
*Yes,for example you can use VirtualBox and Docker as provider,
remember to start the docker service or vagrant will use 
virtualbox as default provider instead of Virtualbox.*
- **It is possible to use a single VM with two providers
different at the same time?**
*No, trying to do so returns an error that explains
that the function will be introduced in later versions.*
- **Convert the Docker image into a Vagrant box?**
*It is possible to do so, [here](https://stackoverflow.com/questions/23436613/how-can-i-convert-a-docker-image-into-a-vagrant-virtualbox-box) there is a tutorial*

