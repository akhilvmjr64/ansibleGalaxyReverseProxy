# Automation of reverse proxy and apache websever setup using ansible roles :
### ARTH Task 15 :
------------------

Here I will show how to install and configure the loadbalancer and apache webserver by creating ansible roles for loadbalancer and apache webserver and then use them and configure the required setup

[Here you can find how to do the complete setup with ansible playbooks](https://github.com/akhilvmjr64/reverseProxy) 

### Creating Ansible role for apache webserver

Firstly to create an ansible role we need to use the command `ansible-galaxy role init <ROLE_NAME>`. After creating the role we need to write the code to install and configure the apache webserver. We know that as ansible role stores the tasks in `tasks/main.yml` file we need to edit this file and write the tasks in it. Following are the tasks that we need to create in `tasks/main.yml` file
- Configure yum(as I am using RedHat as the Operating System)
- Install the httpd software
- Install the php software
- Copy the php code to the server
- Copy the configuration file `httpd.conf`
- Stop the existing service running on the mentioned port
- Start the httpd service
- Configure the firewall to allow the port

To do all the task mentioned above the `tasks/main.yml` looks as follows

```yaml
---
# tasks file for myapache
- name : "configuring yum"
  copy :
    src : "yum.repo"
    dest : "/etc/yum.repos.d/reverseproxy.repo"
- name : "installing httpd package for server"
  package :
    name : "httpd"
    state : present
- name : "installing php package for server"
  package :
    name : "php"
    state : present
- name : "removing the index.html file on webserver"
  file :
    path : "/var/www/html/index.html"
    state : absent
- name : "copying the php file to the webserver"
  copy :
    src : "index.php"
    dest : "/var/www/html/index.php"
- name : "copying the httpd configuration file for the server"
  template :
    src : "httpd.conf"
    dest : "/etc/httpd/conf/httpd.conf"
  notify : "restarting httpd service"
- name : "setting the selinux to permissive mode"
  shell : "setenforce 0"
- name : "killing the service running the {{server_port}} port"
  shell : "kill `netstat -tnlp | grep :{{server_port}} | awk '{print $7}' | awk -F/ '{print $1}'`"
  ignore_errors : yes
- name : "starting the httpd service"
  service :
    name : "httpd"
    state : started
- name : "restarting firewall"
  service :
    name : "firewalld"
    state : restarted
- name : "changing settings for firewall"
  firewalld :
    port : "{{server_port}}/tcp"
    state : enabled
    immediate : yes
```

We can see in this file we have referenced files `yum.repo` in copy task for yum configuration, `index.php` in copy task for copying webpages, `httpd.conf` in template task for configuring httpd server. We need to create files for static files which will be copied to the webserver we use the `files` folder and for those files which uses the ansible variables we use the `templates` folder, in my case I created `yum.repo`, `index.php` in `files` folder and `httpd.conf` in templates folder. To know the contents of these files follow these links [yum.repo](https://github.com/akhilvmjr64/ansibleGalaxyReverseProxy/blob/main/myapache/files/yum.repo), [index.php](https://github.com/akhilvmjr64/ansibleGalaxyReverseProxy/blob/main/myapache/files/index.php), [httpd.conf](https://github.com/akhilvmjr64/ansibleGalaxyReverseProxy/blob/main/myapache/templates/httpd.conf)

We can also see that I have used some variable using templating these variables are created in `vars/main.yml` this file looks as follows
```yaml
---
# vars file for myapache
server_port : 8084
```

Similarly, I have done the configuration for haproxy as follows

### Creating Ansible role for Loadbalancer

Firstly to create an ansible role we need to use the command `ansible-galaxy role init <ROLE_NAME>`. After creating the role we need to write the code to install and configure the haproxy service. We know that as ansible role stores the tasks in `tasks/main.yml` file we need to edit this file and wirte the tasks in it. Following are the tasks that we need to create in `tasks/main.yml` file
- Configure yum(as I am using RedHat as the Operating System)
- Install the haproxy software
- Copy the configuration file `haproxy.cfg`
- Stop the existing service running on the mentioned port
- Start the haproxy service
- Configure the firewall to allow the port

To do all the task mentioned above the `tasks/main.yml` looks as follows

```yaml
---
# tasks file for myloadbalancer
- name : "configuring yum for all the hosts to install httpd, php and haproxy"
  copy :
    src : "yum.repo"
    dest : "/etc/yum.repos.d/reverseproxy.repo"
- name : "installing haproxy on proxy server"
  package :
    name : "haproxy"
    state : "present"
- name : "copying the haproxy configuration file to proxy server"
  template :
    src : "haproxy.cfg"
    dest : "/etc/haproxy/haproxy.cfg"
  notify : "restarting haproxy service"
- name : "setting selinux to permissive mode"
  shell : "setenforce 0"
- name : "restarting firewall"
  service :
    name : "firewalld"
    state : restarted
- name : "killing the service running on {{proxy_port}} port"
  shell : "kill `netstat -tnlp | grep :{{proxy_port}} | awk '{print $7}' | awk -F/ '{print $1}'`"
  ignore_errors : yes
- name : "starting haproxy service"
  service :
    name : "haproxy"
    state : started
- name : "changing firewall settings"
  firewalld :
    port : "{{proxy_port}}/tcp"
    state : enabled
    immediate : yes
```

We can see in this file we have referenced files `yum.repo` in copy task for yum configuration, `haproxy.cfg` in template task for configuring haproxy server. We need to create files for static files which will be copied to the webserver we use the `files` folder and for those files which uses the ansible variables we use the `templates` folder, in my case I created `yum.repo` and `haproxy.cfg` in templates folder. To know the contents of these files follow these links [yum.repo](https://github.com/akhilvmjr64/ansibleGalaxyReverseProxy/blob/main/myloadbalancer/files/yum.repo), [haproxy.cfg](https://github.com/akhilvmjr64/ansibleGalaxyReverseProxy/blob/main/myloadbalancer/templates/haproxy.cfg)

Following is the part of code I have used in the haproxy.cfg file to dynamically add hosts and remove hosts from the list of servers
```cfg
#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     roundrobin
{% for i in groups['servers'] %}
    server  app{{loop.index}} {{i}}:{{server_port}} check
{% endfor %}
```

We can also see that I have used some variable using templating these variables are created in `vars/main.yml` this file looks as follows
```yaml
---
# vars file for myapache
server_port : 8084
proxy_port : 80
```

## Testing these roles
To test these roles we need to create a playbook before creating the playbook we need to write the configuration file for the playbook which can either be created in the present working directory as `ansible.cfg` or in the home directory as `.ansible.cfg`. The ansible configuration file looks as follows
```cfg
[defaults]
inventory=<path-to-inventory>
remote_user=root
host_key_checking=false
role_path=<path-to-roles>
```
To test the roles created above I have created `main.yml` file which looks as follows
```yaml
- hosts : servers
  roles :
          - myapache
- hosts : proxy
  roles :
          - myloadbalancer
```
**Note : In the above file I have used the group name these are the only group names that must be used if you are using the same code what I have created**

To run the playbook we need to use the command `ansible-playbook main.yml`

## [You can find the article on the same here](https://burriakhilreddy.hashnode.dev/ansiblegalaxyreverseproxy)
