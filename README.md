Ansible Sendy role
=====

This role installs and configures Sendy on a server.

Requirements
------------

This role requires Ansible 1.4 or higher and platform requirements are listed
in the metadata file.

Role Variables
--------------

The variables that can be passed to this role and a brief description about
them are as follows.

```yaml
# version to install
sendy_version: "2.0.7"

# get sendy via url
sendy_archive_with_get_url: true
sendy_get_url_src: "http://url_to_get_sendy/sendy-{{sendy_version}}.zip"
sendy_get_url_username:
sendy_get_url_password:

# sendy's destination
sendy_dest_path: "/usr/share/nginx/html"
sendy_full_dest_path: "{{sendy_dest_path}}/sendy"
sendy_user: "www-data"
sendy_group: "www-data"

# database information
sendy_db_name: 'sendy'
sendy_db_host: 'localhost'
sendy_db_port: 3306
sendy_db_username: 'tbd'
sendy_db_password: 'tbd'
sendy_db_permissions: "{{sendy_db_name}}.*:ALL"

# sendy's configuration
sendy_config_app_path: 'http://your_sendy_installation_url'
sendy_config_app_cookie_domain: ''
sendy_config_app_charset: 'utf8'
```

Examples
========

```yaml
- name: emailing
  hosts: emailing
  user: root
  roles:
    - role: nbz4live.php-fpm
      fpm_pools:
        - pool:
            name: default
            vars:
              - user = www-data
              - group = www-data
              - listen = "/var/run/php5-fpm.sock"
              - listen.owner = www-data
              - listen.group = www-data
              - listen.mode = 0660
              - pm = dynamic
              - pm.max_children = 75
              - pm.start_servers = 2
              - pm.min_spare_servers = 2
              - pm.max_spare_servers = 5
              - pm.max_requests = 50
              - pm.status_path = '/fpm_status'
              - chdir =  /
              - slowlog = '/var/log/php5-fpm.log.slow'
              - request_slowlog_timeout = 1s
      php_config:
        - option: "date.timezone"
          section: "PHP"
          value: "Etc/UTC"
        - option: "engine"
          section: "PHP"
          value: "1"
        - option: "error_reporting"
          section: "PHP"
          value: "E_ALL & ~E_DEPRECATED & ~E_STRICT"
      php_fpm_default_pool:
        delete: yes
        name: default.conf
    - role: jdauphant.nginx
      nginx_sites:
        sendy:
          - listen "80"
          - server_name {{sendy_fqdn}}
          - root {{sendy_full_dest_path}}
          - index index.php
          - access_log /var/log/nginx/{{sendy_fqdn}}_access.log nm-combined
          - error_log /var/log/nginx/{{sendy_fqdn}}_error.log
          - autoindex off
          - location / {
              try_files $uri $uri/ $uri.php?$args;
            }
          - location /l/ {
              rewrite ^/l/([a-zA-Z0-9/]+)$ /l.php?i=$1 last;
            }
          - location /t/ {
              rewrite ^/t/([a-zA-Z0-9/]+)$ /t.php?i=$1 last;
            }
          - location /w/ {
              rewrite ^/w/([a-zA-Z0-9/]+)$ /w.php?i=$1 last;
            }
          - location /unsubscribe/ {
              rewrite ^/unsubscribe/(.*)$ /unsubscribe.php?i=$1 last;
            }
          - location /subscribe/ {
              rewrite ^/subscribe/(.*)$ /subscribe.php?i=$1 last;
            }
          - location ~ \.php$ {
              try_files $uri =404;
              fastcgi_split_path_info ^(.+\.php)(/.+)$;
              fastcgi_pass unix:{{fpm_sock_path}};
              fastcgi_index index.php;
              fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
              include fastcgi_params;
            }
    - deimosfr.sendy
```

Dependencies
------------

None

License
-------

GPL

Author Information
------------------

Pierre Mavro / deimosfr
