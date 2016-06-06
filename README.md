#http-haproxy

Setup haproxy as reverse http(s)-proxy with no pain. 

Just define the domain-names and the backends ip's to configure a reverse proxy.

#Features

- Easy to configure
- ACL's 
- Domain-name or SNI based backend selection
- Enforce redirects http -> https
- "Routing": different url's can connect to different backends
- Special Route for letsecnrypt.org (since 0.4)
- Template for galera-cluster (since 0.4)

#Role Variables

##Named acl's `haproxy_source_acl`

A dictionary containing { acl-name : list with ip-ranges } which can be used for access control in frontends.

Example:

    haproxy_source_acl:
    github.com: 
    - 192.30.252.0/22
    office.public:
    - 88.66.77.0/24
    private.range:
    - 127.0.0.0/8
    - 10.0.0.0/8
    - 192.168.0.0/16
    - 172.16.0.0/12
  

##Domains `haproxy_domains`

A dictionary containing { domain-name : config } 

Minimal config:

    haproxy_domains:
      www.example.com:
        backend:
          servers:
          - 10.100.2.60

All config options:

    haproxy_domains:
      jenkins.example.com:
        https: true         # optional default is false
        http: true          # optional default is true
        redirect: false     # optional default is false, if true https and http must be true to redirect domain to https
        options:            # optional, default is balance leastconn, option httpclose, option forwardfor
        - key: value        # if used: list with key: value pairs
        limit_to:           # optional, no default
        - office.public     # if used: list names defined in "haproxy_source_acl"
        - github.com
        backend:
          port: 8080        # optional -> default is 80
          check:            # optional -> default is check 
          servers:
          - 10.100.2.80
          - 10.100.2.81
          - 10.100.2.82

##Enable statistics frontend `haproxy_stats_users`

To enable haproxys statistic frontend, just define users.

    haproxy_stats_users:
    - {username: admin, password: haproxy}

Change statistics URL with:

    haproxy_stats_uri: '/haproxy?stats'

##Bind address `haproxy_default_bind_address`

By default haproxy listens to all interfaces (*). You can override it by setting:

    haproxy_default_bind_address: 23.4.5.6

##SSL support

To enable ssl (https) you need certificates stored in .pem format, including all intermediate- and root-certs.

The name of the file must be _domain-name_.pem (_domain-name_ must match a key in `haproxy_domains`).
And the file must be stored in the directory defined in `haproxy_pem_lookup` (Default is ./certs/ relative to your inventory file )

##Multiple frontends

Since version 0.2 it's possible to have multiple frontends running with different ip's.

    haproxy_frontends:
      internal: 
        bind_address: 10.100.2.30
      public: 
        bind_address: 10.100.2.33
      another: 
        bind_address: 10.100.2.34  

Bind domains to frontends by adding `frontends` to domain config. If no frontend is defined for a domain it will listen on the first one defined in `haproxy_frontends`

    haproxy_domains:
    jenkins.example.com:
      ....
      ....
      frontends:
        - public
        - another
      ....

##Routes

Since version 0.3

Use different backends based on request url.

Example:

    seafile.proxy:
      backend:
        port: 8000
        servers:
        - 10.100.2.91
        routing:
          seafhttp:
            striproute: True      # optional, if set to True the url will be rewritten ( /seafhttp/ to / in this example )
            backend:
              port: 8082
              servers:
              - 10.100.2.91

##Letsencrypt 

Since version 0.4

If you want to use letsencrypt.com to create ssl certificates set `haproxy_letsencrypt_validation` to true.

This will create an path (/.well-known/acme-challenge) based acl.

Request matching this acl will be redirected to the host defined in `haproxy_letsencrypt_validation_server`

For convenience the url's are rewritten:

    http://example.com/.well-known/acme-challenge/111
    will be rewriten to {{haproxy_letsencrypt_validation_server}}/example.com/.well-known/acme-challenge/111 


##Galera cluster

Since version 0.4

Create a haproxy listen-config for a MySQL cluster (@see templates/galera-cluster.j2)

Enable template by setting `haproxy_mysql_cluster` to True.

List all MySQL nodes in `haproxy_mysql_nodes`

Set ip and port with `haproxy_mysql_listen_ip` and `haproxy_mysql_listen_port`

There are two check methods: mysql or http (`haproxy_mysql_check_type`)
- mysql : check connection directly via mysql-port
- http : check mysql over xinetd -> script (@see https://github.com/netzwirt/ansible-galera-cluster)

`haproxy_mysql_check_port`is onnly required when check method http is used.


##TCP listeners

Since version 0.5 

Added `options` in version 0.6.

Simple tcp forwarding:

Config example (forward 10.10.10.10:2022 to 22.33.44.55:22):

    haproxy_tcp_listen:
    - name: sftp-server
      bind_address: 10.10.10.10
      bind_port: 2022
      options:
      - 'server timeout 60000'
      - 'client timeout 120000'
      backend:
        port: 22
        check: ''
        servers:
        - 22.33.44.55


#Dependencies

None

#Example Playbook

This is an example playboox with the minimal required config. It will setup haproxy for www.example.com listening on port 80 with 2 backend-servers (10.100.2.62,10.100.2.61).

    - hosts: haproxy
      become: true
      vars:
        haproxy_domains:
          www.example.com:
            backend:
              servers:
              - 10.100.2.60
              - 10.100.2.61
      roles:
         - { role: netzwirt.http-haproxy }


#License

BSD

#Author Information

[netzwirt](https://github.com/netzwirt)

