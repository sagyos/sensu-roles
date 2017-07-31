[![ansible-uchiwa](https://travis-ci.org/queeno/ansible-uchiwa.png?branch=master)](https://travis-ci.org/queeno/ansible-uchiwa)

# Ansible-Uchiwa

Ansible role for installing and configuring [Uchiwa](https://uchiwa.io), the open-source sensu dashboard.

## Requirements

This role doesn't have any hard dependencies on other roles. Uchiwa can run on a standalone, vanilla box.
However, you may want to connect Uchiwa to sensu-api in order to display clients and events.

You can install Sensu and RabbitMQ on the same node. The following Ansible roles may help:

- [ansible-playbook-rabbitmq](https://github.com/Mayeu/ansible-playbook-rabbitmq)
- [ansible-playbook-sensu](https://github.com/Mayeu/ansible-playbook-sensu)

## Supported systems

Currently Debian-like and RedHat-like operating systems are supported. This includes the popular Ubuntu and CentOS.

This role is tested on ubuntu/precise64 and centos/6.5 OSes. For more information, check out the vagrant folder included in this project.

## Installation

You can use [ansible-galaxy](https://galaxy.ansible.com/list#/roles/2192):

```
$ ansible-galaxy install queeno.uchiwa
```

## Role default variables

These variables are defined in `defaults/main.yml`. Please overwrite if you have different requirements.


|Name                         |Type             |Description                                   |Default                         |
|-----------------------------|-----------------|----------------------------------------------|--------------------------------|
|`supported_os_families`      |List of strings  |Supported OS families by the role             |['Debian','RedHat']             |
|`install_repo`               |Boolean          |Determines whether to install sensu repo      |True                            |
|`sensu_apt_repo_url`         |String           |Sensu APT repository URL                      |'http://repos.sensuapp.org/apt' |
|`sensu_apt_repo_distribution`|String           |Sensu APT repository distribution             |'sensu'                         |
|`sensu_apt_repo`             |String           |Sensu APT repository name                     |'main'                          |
|`sensu_apt_repo_key_url`     |String           |Sensu APT repository key URL                  |Please see <sup>1</sup>|
|`sensu_apt_repo_key_id`      |String           |Sensu APT repository key ID                   |'7580C77F'                      |
|`sensu_yum_repo`             |String           |Determines which YUM sensu repo to install <sup>2</sup> |'sensu'                         |
|`version`                    |String           |Determines the uchiwa version to install <sup>3</sup> |'installed'                     |
|`uchiwa_apt_pkg`             |String           |The uchiwa package name for Debian systems    |'uchiwa'                        |
|`uchiwa_yum_pkg`             |String           |The uchiwa package name for RedHat systems    |'uchiwa'                        |
|`sensu_api_endpoints`        |List of hashes   |List of sensu API endpoint configuration <sup>4</sup> |Please see <sup>4</sup>                 |
|`host`                       |String           |The uchiwa host or IP address to bind uchiwa to |'0.0.0.0'                     |
|`port`                       |Integer          |The port uchiwa listens on                    |3000                            |
|`user`                       |String           |The username to access uchiwa dashboard       |'' *(empty for none)*              |
|`pass`                       |String           |The password to access uchiwa dashboard       |'' *(empty for none)*              |
|`refresh`                    |Integer          |The interval to pull the sensu APIs (in seconds)|5                             |
|`users`                      |List of hashes   |Multiple user credentials to access uchiwa dashhboard|Please see <sup>5</sup>      |
|`auth`                       |Hash             |Contains the path to the private and public keys for JWT validation|Please see <sup>6</sup>      |

### Footnotes

<sup>1</sup> `http://repos.sensuapp.org/apt/pubkey.gpg`

<sup>2</sup> `sensu_yum_repo` could be either `sensu` (installs the stable YUM repo) or `sensu-unstable` (installs the unstable YUM repo). The repo configuration is in `files\*sensu_yum_repo*.repo`.

<sup>3</sup> `version` takes the following values:

- `latest`: Ensures the uchiwa latest version is always installed.
- `installed`: Installs the uchiwa latest version without updating it.
- `*version number*`: Explicitly specify a version number to pin uchiwa to a specific version.
	
<sup>4</sup> `sensu_api_endpoints` identifies the sensu API endpoints, so uchiwa can connect to them.

You don't need to overwrite all the hash keys in the array. If you don't pass a hash key, the respective default will be used.
	
|Name        |Type         |Description       |Default            |
|------------|-------------|------------------|-------------------|
|`name`      |String       |The name of the sensu API endpoint | 'sensu' |
|`host`      |String       |The IP address or name of the sensu API endpoint | '127.0.0.1'  |
|`ssl`       |Boolean      |Determines whether or not the sensu API endpoint requires SSL authentication | False    |
|`insecure`  |Boolean      |Determines whether or not to accept an insecure SSL certificate  | True        |
|`port`      |Integer      |The sensu API endpoint listening port    | 4567     |
|`user`      |String       |The username of the sensu API endpoint        | '' *(empty for none)* |
|`pass`      |String       |The password of the sensu API endpoint        | '' *(empty for none)* |
|`path`      |String       |The path to the sensu API endpoint            | '' *(empty for none)* |
|`timeout`   |Integer      |Timeout of the sensu API endpoint in seconds  | 5                     |

<sup>5</sup> `users` specifies a list of user credentials to access the uchiwa dashboard. This feature is available from uchiwa 0.10.0.
`users` has precedence over `user`. If left empty or omitted the `user` and `password` values will be used instead.

Please specify the following keys for each element of `users`:

|Name        |Type         |Description       |Default            |
|------------|-------------|------------------|-------------------|
|`username`  |String       |The username to access uchiwa dashboard   |   |
|`password`  |String       |The password to access uchiwa dashboard   |   |
|`readonly`  |Boolean      |Should the user have read only access?    |false  |

<sup>6</sup> `auth` specifies the public and private keys for JTW validation and signing. If this hash is omitted, then uchiwa will generate its own keys on start behind the scenes. For this reason, all fields are mandatory if variable is declared.

Please populate the `auth` hash with the following values:

|Name       |Type          |Description       |Default            |
|-----------|--------------|------------------|-------------------|
|`privatekey`|String       |Path to the private key|              |
|`publickey` |String       |Path to the public key |              |

## Files required

You need to have the following files in the role folder (for RedHat only):

```
 files/
  | - sensu.repo
  | - sensu-unstable.repo
```

## Dependencies

No hard dependency.

Weak dependencies on:

- Redis
- RabbitMQ
- Sensu
 
## Tests

### Vagrant

Vagrant creates a `Debian`-like VM (hashicorp/precise64) and a `RedHat`-like VM (chef/centos-6.5).
The `vagrant up` command will trigger a `standalone.yml` ansible-playbook run that will test this role on the two VMs.

You can also spin up a specific vm:

`vagrant up debian` spins up *hashicorp/precise64*
`vagrant up redhat` spins up *chef/centos-6.5*

### Acceptance Tests

The acceptance tests will be in the `vagrant/` folder and will be integrated with `travis-ci`. Watch this space for more info.

## Licence

BSD
