# rancher-demonstration

This repository is a simple demonstration of virtualized environments
([vagrant](https://www.vagrantup.com/) + [virtualbox](https://www.virtualbox.org/)),
tailored to exhibit the [kubernetes](https://kubernetes.io/)-based
[rancher-server](https://github.com/rancher/rancher), and [rancher-agent](https://github.com/rancher/agent) ecosystem.
Specifically, a [`Vagrantfile`](https://github.com/jeff1evesque/rancher-demonstration/blob/master/Vagrantfile) will automate the processes contained within
the provided [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility).

In this setup, the [rancher-server](https://github.com/jeff1evesque/rancher-demonstration/blob/master/utility/install-rancher-server)
is launched using [Centos7x](https://github.com/jeff1evesque/rancher-demonstration/blob/1959f5817ca53d89c8d3349d3bb23406c3bf3ea6/Vagrantfile#L40-L46),
while the [rancher-agent](https://github.com/jeff1evesque/rancher-demonstration/blob/master/utility/install-rancher-agent)
is launched in [Bionic64](https://github.com/jeff1evesque/rancher-demonstration/blob/1959f5817ca53d89c8d3349d3bb23406c3bf3ea6/Vagrantfile#L47-L53).
If the rancher-server needs to be debian-based, or corresponding rancher-agents
need to be rhel-based, then additional utility scripts, not included in this
repository, will need to be created.

Regardless of implementation, when vagrant completes provisioning, a rancher-server,
is available via https://localhost:7895, and can be used to manage various
[kubernetes](https://kubernetes.io/)-based clusters:

![Rancher Login](https://user-images.githubusercontent.com/2907085/51079846-69126300-169d-11e9-9c06-6da88c38a0df.PNG)

---

Upon immediate login, the rancher-server may still be provisioning:

![Rancher Provisioning](https://user-images.githubusercontent.com/2907085/51079851-a5de5a00-169d-11e9-8ee0-087483ffbff0.PNG)

**Note:** the provided [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility),
used to provision the corresponding vagrant virtual machine(s), can also be
used on production-like environments. However, for [high-availability](https://rancher.com/docs/rancher/v2.x/en/installation/ha/),
the [install-rancher-server](https://github.com/jeff1evesque/rancher-demonstration/blob/master/utility/install-rancher-server)
will need to be adjusted.

---

After a few minutes, the cluster will complete provisioning, and be active:

![Rancher Active](https://user-images.githubusercontent.com/2907085/51079860-cd352700-169d-11e9-859a-5dc6ce9f6d39.PNG)

---

Then, the newly created cluster can be reviewed:

![Rancher Cluster](https://user-images.githubusercontent.com/2907085/51079869-e8a03200-169d-11e9-96c6-457ec62fb695.PNG)

## Additional hosts

Additional hosts can be added to a desired cluster, by first clicking edit
under the top-right hamburger icon:

![rancher-add-host](https://user-images.githubusercontent.com/2907085/51116143-446ed600-17d8-11e9-8642-97d82fc4d36e.PNG)

At the bottom of the associated page, the corresponding docker command can be
pasted into the desired cluster host:

![rancher-new-host-command](https://user-images.githubusercontent.com/2907085/51116220-8566ea80-17d8-11e9-9c8d-f0d223cb697f.PNG)

## Deployment

The `kubectl` command can be executed on the rancher-server via the web-browser,
or directly within the container `docker exec -it rancher-server /bin/bash`.

```bash
[root@rancher-server vagrant]# docker ps -a
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                                                               NAMES
92dde0d45453        rancher/rancher:v2.1.5   "entrypoint.sh"     38 hours ago        Up 38 hours         0.0.0.0:8890->80/tcp, 0.0                      .0.0:8895->443/tcp   rancher
[root@rancher-server vagrant]# docker exec -it rancher /bin/bash
root@92dde0d45453:/var/lib/rancher# kubectl run \
  --image=nginx \
  --port=80 \
  --env='DOMAIN=cluster' \
  replicas=3
```

**Notes:** if `--env=` is passed, environment variables can be read from STDIN
using the standard env syntax.

Additionally, a yaml configuration file can be utilized:

```bash
[root@rancher-server vagrant]# docker ps -a
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                                                               NAMES
92dde0d45453        rancher/rancher:v2.1.5   "entrypoint.sh"     38 hours ago        Up 38 hours         0.0.0.0:8890->80/tcp, 0.0                      .0.0:8895->443/tcp   rancher
[root@rancher-server vagrant]# docker exec -it rancher /bin/bash
root@92dde0d45453:/var/lib/rancher# kubectl create -f ./manifest.yaml
root@92dde0d45453:/var/lib/rancher# kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           18s
```

The following is an example `manifest.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
          - containerPort: 80
        env:
        - name: DOMAIN
          value: cluster
```

**Note:** for development purposes, the `kind: Pod` can be implemented if
`replicas` are not needed. However, in production systems, the `kind: Deployment`
is typically desired.

**Note:** [Kompose](http://kompose.io/user-guide/) can convert an existing
`docker-compose.yml` to a series of kubernetes yaml files.

Alternatively, instead of defining multiple environment variables for each yaml
file, a custom [`configMapRef`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)
can be created:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  DOMAIN: cluster
```

This allows the above `manifest.yaml` to refactor as follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
          - containerPort: 80
        envFrom:
          - configMapRef:
            name: special-config
            key: DOMAIN
```

**Note:** if the `envFrom` > `key` is omitted, the entire `special-config`,
will be available within the given container.

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
command to build the rancher-server:

```bash
cd /path/to/rancher-demonstration/
vagrant up
```

**Note:** an alternative syntax to `vagrant up`, is to run `vagrant up rancher-server`.
However, the associated `rancher-agent` needs to be created as well.

Depending on the network speed, the build can take between 2-4 minutes. So,
grab a cup of coffee, and perhaps enjoy a danish while the virtual machine
builds.

**Note:** a more complete refresher on virtualization, can be found within the
vagrant [wiki page](https://github.com/jeff1evesque/machine-learning/wiki/Vagrant).

Though, the implemented [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility)
can be used to provision vagrant, it can be run on non-vagrant systems. For example,
the [install scripts](https://github.com/jeff1evesque/rancher-demonstration/tree/master/utility)
can easily be run on corresponding virtual machines:

```bash
## virtual machine for rancher-server
./install-rancher-server \
  "$server_version" \
  "$server_internal_port" \
  "$server_internal_https_port" \
>> "${project_root}/logs/install-rancher-server.txt" 2>&1

## virtual machine for rancher-agent
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
dependencies have been accounted.