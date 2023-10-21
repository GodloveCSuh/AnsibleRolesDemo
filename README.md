# Ansible Roles

Ansible Roles are reusable and modular units in Ansible. They simplify the organization and management of Ansible playbooks by bundling related tasks, variables, and templates into a single directory structure. Roles make it easier to implement consistent configurations and automation across multiple hosts. They encapsulate roles, reducing code duplication, and promoting a structured approach to automation. Roles typically include directories for tasks, handlers, templates, variables, and more, making it simple to manage complex infrastructure tasks. By using roles, Ansible users can enhance the scalability, maintainability, and reusability of their automation code, streamlining configuration management and orchestration.

## Goal

In this example, we'll use the Nginx web server and include features like templates and handlers. This role will install Nginx, deploy a basic website, and manage the Nginx service.

Create a new Ansible role named `webserver`:

```bash
ansible-galaxy init ./roles/webserver
```

## **Define Role defaults variable**

Defaults are used to define default variables that can be overridden by users or other roles. These variables provide flexibility and customization for the role's behavior.
Location: Default variables are defined in the defaults/main.yml file within the role directory.
Syntax: Variables are defined as key-value pairs in the defaults/main.yml file. Users can override these variables in their playbook or inventory files.

Edit roles/webserver/defaults/main.yml to define default variables:

```yaml
---
nginx_user: www-data
nginx_worker_processes: auto

nginx_error_log: /var/log/nginx/error.log
nginx_pid: /var/run/nginx.pid

nginx_worker_connections: 1024

nginx_server_name: localhost
nginx_root_dir: /var/www/html
nginx_index_file: index.html

```

## **Define Role Tasks**

Edit `roles/webserver/tasks/main.yml` to define the tasks:

```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present
  notify:
    - Restart Nginx

- name: Copy Nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - Restart Nginx

- name: Deploy Website
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
  notify:
    - Restart Nginx

- name: Ensure Nginx is enabled and running
  service:
    name: nginx
    enabled: yes
    state: started
```

## **Define Role Handlers**

Purpose: Handlers are used to define tasks that should be triggered when specific events occur during the playbook run. Common use cases include restarting a service after a configuration change or reloading a web server after deploying new content.
Location: Handlers are defined in the handlers/main.yml file within the role directory.
Syntax: Handlers are written as named tasks under the handlers section. They are typically associated with specific notify statements in task files.
Example: In the role's handlers/main.yml file, you might define a handler named "Restart Nginx" to restart the Nginx service.

Edit the roles/webserver/handlers/main.yml file:

```yaml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted

```

## **Define Role Templates**

Create template files for Nginx configuration and the website content.

In the `roles/webserver/templates` directory, create `nginx.conf.j2`:

```jinja2
user {{ nginx_user }};
worker_processes {{ nginx_worker_processes }};
error_log {{ nginx_error_log }};
pid {{ nginx_pid }};

events {
    worker_connections {{ nginx_worker_connections }};
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 80;
        server_name {{ nginx_server_name }};

        location / {
            root {{ nginx_root_dir }};
            index {{ nginx_index_file }};
        }
    }
}
```

In the same directory, create `index.html.j2`:

```jinja2
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to My Website</title>
</head>
<body>
    <h1>Hello, World from Chongwain!</h1>
</body>
</html>
```

## **Include the Role in a Playbook**

In your playbook, include the `webserver` role. Create or edit your `playbook.yml`:

```yaml
---
- name: Configure Web Server
  hosts: web_servers
  become: yes
  roles:
    - webserver
```

## **Run the Playbook**

Run the playbook using the `ansible-playbook` command:

```bash
ansible-playbook -i inventory/hosts.ini playbook.yml
```

## Conclusion

This Ansible role will install Nginx, deploy a simple website, and manage the Nginx service. It showcases the use of templates for configuration files and handlers for service restarts. Adjust the content of the templates and other tasks to suit your specific web server configuration needs.
