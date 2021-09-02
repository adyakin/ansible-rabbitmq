Rabbitmq 
========

Playbook that installs and configures RabbitMQ message broker.

Supports standalone or simple cluster deployment, it is based on https://github.com/abelboldu/ansible-rabbitmq role, without SSL or federation support.


## Role Variables

### Example inventory file

```yaml
rabbitmq:
  hosts:
    172.16.128.121:
    172.16.128.122:
    172.16.128.123:
  vars:
    rabbitmq_cluster: true
    rabbitmq_cluster_name: r1
    rabbitmq_ssl: yes
    rabbitmq_ssl_cluster: yes
    rabbitmq_plugins:
      - rabbitmq_management
      - rabbitmq_federation
      - rabbitmq_federation_management
      - rabbitmq_prometheus
      - rabbitmq_shovel
      - rabbitmq_shovel_management
```

### Environment

Use `rabbitmq_conf_env` so set Environment variables such as NODENAME,
HOSTNAME, RABBITMQ_USE_LONGNAME, NODE_PORT, NODE_IP_ADDRESS, etc.

Example:

```yaml
rabbitmq_conf_env:
  NODENAME: rabbit1

```

### Configuration file

`rabbitmq_tcp_address` - listening address for the tcp interface, such
as `0.0.0.0`.

`rabbitmq_tcp_port` - listening port for the tcp interface, such as `5672`.

`rabbitmq_cluster` - a boolean variable, when set to `True` the role will add
all nodes in a play group to a cluster setup in a configuration file. It
depends on a `ansible_play_hosts` magic variable, found in ansible 2.2
or later.

`rabbitmq_cluster_name` - cluster name, default is `rabbit`

`rabbitmq_erlang_cookie` - only used when `rabbitmq_cluster` is used, to
identify members of a single cluster.

`rabbitmq_ssl` - boolean, enable ssl for amqp, default - `no`. If set role loads variable `vars/{{ rabbitmq_cluster_name }}-ssl.yml` which should contains `certificates` dict with following objects:
 - `ca` - Root CA certificate
 - `{{ ansible_hostname|lower }}.crt`, `{{ ansible_hostname|lower }}.crt` - certificate and key for each host.

`rabbitmq_ssl_cluster` - boolean, enable ssl for inter-node connectivity, default - `no`. Uses same certificates from `certificates` variable.


### Plugins

`rabbitmq_plugins` - list of plugins to activate.

### Users

:bangbang: | For multiple cluster configuration see https://github.com/adyakin/ansible-rabbitmq-configuration
:---: | :---

`rabbitmq_users` - list of users, and associated vhost and password.
Example on defining the users configuration:

```yaml
rabbitmq_users:
  - user:     user1
    password: password1             # Optional, defaults to ""
    vhost:    vhost1                # Optional, defaults to "/"
    node:     node_name             # Optional, defaults to "rabbit"
    configure_priv: "^resource.*"   # Optional, defaults to ".*"
    read_priv: "^$"                 # Disallow reading (defaults to ".*")
    write_priv: "^$"                # Disallow writing (defaults to ".*")
  - user:     user2
    password: password2
    vhost:    vhost1
    force:    no
    tags:                           # Optional, user tags
    - administrator
  - user:     guest
    state:    absent                # Optional, removes user (defaults to "present")
```

### Vhosts

`rabbitmq_vhosts` - list of vhosts to create. Example on defining the
vhosts configuration:

```yaml
rabbitmq_vhosts:
  - name:     vhost1
    node:     node_name             # Optional, defaults to "rabbit"
    tracing:  yes                   # Optional, defaults to "no"
    state:    present               # Optional, defaults to "present"
```

### Policies

`rabbitmq_policies` - list of policies to be created (or removed if
`state: absent` is set). Example on defining the policies configuration:

```yaml
rabbitmq_policies:
  - name:     HA Policy
    vhost:    '/'                   # Optional, defaults to "/"
    pattern:  '.*'                  # Optional, defaults to ".*"
    tags:                           # Optional, defaults to "{}"
      ha-mode: all
      ha-sync-mode: automatic
    state:    present               # Optional, defaults to "present"
```

### Cluster setup

To create cluster `rabbitmq_cluser` variable should be `yes`. First node in play considered as master node. 

### File descriptors

`rabbitmq_fd_limit` - set this to some numeric value to override 1024
default. Currently only supports systemd.

## Testing

There is some tests that try to provision a VM using Vagrant. Just launch them
with:

    $ cd tests
    $ vagrant up



## License

BSD
