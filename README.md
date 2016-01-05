#http-haproxy

Setup haproxy as reverse http(s)-proxy with no pain. 

Just define the domain-names and the backends ip's to configure a reverse proxy.

#Features

- Easy to configure
- ACL's 
- Domain-name or SNI based backend selection
- Enforce redirects http -> https

#Role Variables

##Named acl's `haproxy_source_acl`

A dictionary containing { acl-name : list with ip-ranges } which can be used for access control in the frontend.

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

