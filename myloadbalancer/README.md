HAProxy
=======

## Role for Installation of `HaProxy`
With this role one can install `Haproxy` and automatically configure it

Role Variables
--------------

- ### `server_port`
	is the variable for storing the port number for the webserver
- ### `proxy_port`
	is the variable for storing the port number for the haproxy


Example Playbook
----------------
```yaml
    - hosts: proxy
      roles:
         - haproxy
```
