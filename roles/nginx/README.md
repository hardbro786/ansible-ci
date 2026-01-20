# NGINX Ansible Role

This role installs and configures NGINX and serves a simple index.html page.

## Variables
- `nginx_port` (default: 80)
- `nginx_user` (default: www-data)

## Example Playbook
```yaml
- hosts: web
  become: yes
  roles:
    - nginx

