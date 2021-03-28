HTTPD
=====

## Role for Installation of `httpd`
With this role one can install `httpd` and automatically configure it

Role Variables
--------------

- ### `server_port`
	is the variable for storing the port number for the webserver

Example Playbook
----------------
```yaml
    - hosts: servers
      roles:
         - myapche
```
