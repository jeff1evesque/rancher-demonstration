# rancher-demonstration

This repository is a simple demonstration of virtualized environments
 ([vagrant](https://www.vagrantup.com/) + [virtualbox](https://www.virtualbox.org/)),
 tailored to exhibit the [rancher-server](https://github.com/rancher/rancher)
 / [rancher-agent](https://github.com/rancher/agent) ecosystem.
 Specifically, a [`Vagrantfile`](https://github.com/jeff1evesque/rancher-demonstration/blob/master/Vagrantfile) will automate the processes contained within
 the provided [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility).

In this setup, the [rancher-server](https://github.com/jeff1evesque/rancher-demonstration/blob/master/utility/install-rancher-server)
 is launched using [Centos7x](https://github.com/jeff1evesque/rancher-demonstration/blob/1959f5817ca53d89c8d3349d3bb23406c3bf3ea6/Vagrantfile#L40-L46),
 while the [rancher-agent](https://github.com/jeff1evesque/rancher-demonstration/blob/master/utility/install-rancher-agent)
 is launched in [Xenial64](https://github.com/jeff1evesque/rancher-demonstration/blob/1959f5817ca53d89c8d3349d3bb23406c3bf3ea6/Vagrantfile#L47-L53).
 If the rancher-server needs to debian-based, or corresponding rancher-agents
 need to be rhel-based, then additional utility scripts will need to be created.

Regardless of implementation, when vagrant completes provisioning, a rancher-server,
 is available via https://localhost:7895, and can be used to manage various
 [kubernetes](https://kubernetes.io/) based clusters:

![Rancher Login](https://user-images.githubusercontent.com/2907085/51079846-69126300-169d-11e9-9c06-6da88c38a0df.PNG)

---

Upon immediate login, we may notice the rancher-server still be provisioning:

![Rancher Provisioning](https://user-images.githubusercontent.com/2907085/51079851-a5de5a00-169d-11e9-8ee0-087483ffbff0.PNG)

**Note:** the provided [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility),
 used to provision the corresponding vagrant virtual machine(s), can also be
 used on production-like environments, just remember to replace the default ssl
 keys, along with other system specific configurations.

---

After a few minutes, the cluster will complete provisioning, and active:

[Rancher Active](https://user-images.githubusercontent.com/2907085/51079860-cd352700-169d-11e9-859a-5dc6ce9f6d39.PNG)

---

Then, the newly created cluster can be reviewed:

[Rancher Cluster](https://user-images.githubusercontent.com/2907085/51079869-e8a03200-169d-11e9-96c6-457ec62fb695.PNG)

## Configuration

Fork this project in your GitHub account.  Then, clone your repository, with
 one of the following approaches:

- [simple clone](https://jeff1evesque.github.io/machine-learning.docs/latest/html/configuration/setup-clone#simple-clone):
 clone the remote master branch.
- [commit hash](https://jeff1evesque.github.io/machine-learning.docs/latest/html/configuration/setup-clone#commit-hash):
 clone the remote master branch, then checkout a specific commit hash.
- [release tag](https://jeff1evesque.github.io/machine-learning.docs/latest/html/configuration/setup-clone#release-tag):
 clone the remote branch, associated with the desired release tag.

## Installation

In order to proceed with the installation for this project, three dependencies
 need to be installed:

- [Vagrant](https://www.vagrantup.com/)
- [Virtualbox x.y.z](http://download.virtualbox.org/virtualbox/5.1.2/) (or higher)
- [Extension Pack x.y.z](http://download.virtualbox.org/virtualbox/5.1.2/) (required)

Once the necessary dependencies have been installed, execute the following
 command to build the puppetserver:

```bash
cd /path/to/rancher-demonstration/
vagrant up
```

**Note:** an alternative syntax to `vagrant up`, is to run `vagrant up rancher-server`.
 However, the associated `rancher-agent` needs to be created as well.

Depending on the network speed, the build can take between 10-15 minutes. So,
 grab a cup of coffee, and perhaps enjoy a danish while the virtual machine
 builds.

**Note:** a more complete refresher on virtualization, can be found within the
 vagrant [wiki page](https://github.com/jeff1evesque/machine-learning/wiki/Vagrant).

Though, the implemented [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility)
 can be used to provision vagrant, it can be run on non-vagrant systems. For example,
 the [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility)
 can easily be run on corresponding virtual machines:

```bash
## virtual machine for puppetserver
./install-rancher-server \
  "$server_version" \
  "$server_internal_port" \
  "$server_internal_https_port" \
>> "${project_root}/logs/install-rancher-server.txt" 2>&1

## virual machine for puppetagent
./install-rancher-agent \
  "$agent_version" \
  "$server_ip" \
  "$server_internal_port" \
  "$server_internal_https_port" \
>> "${project_root}/logs/install-rancher-agent.txt" 2>&1
```

**Note:** when running the above [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility),
 it is assumed that [docker](https://github.com/jeff1evesque/rancher-demonstration/blob/master/utility/install-docker),
 and [rancher-cli](https://github.com/jeff1evesque/rancher-demonstration/blob/master/utility/install-rancher-cli)
 dependencies have been acounted.
