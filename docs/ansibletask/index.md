# Automate with ansible
## Description
Attempted to automate the deployment of Django and Laravel projects using the Nginx web server on different Linux distributions.

### Inventory
The inventory.ini file in Ansible is used to define the list of hosts or groups of hosts that Ansible will manage. It can include hostnames, IP addresses, and group names, as well as variables like SSH ports, user credentials, and specific configurations for each host. Hosts are categorized in sections, such as [webservers] or [databases], making it easier to organize and target systems in playbooks.

```
[laravel:children]
vm1
vm2

[django:children]
vm1
vm3

[router:children]
vm1

[NAT:children]
vm2
vm3

[vm1]
192.168.55.100 ansible_user=ubuntu1 bridgedInterface=ens160 server_name='192.168.55.100 192.168.1.115 ubuntu1.ir www.ubuntu1.ir' DNS_Address='nameserver 10.202.10.10' ALLOWED_HOSTS='ALLOWED_HOSTS=["192.168.55.100", "192.168.1.115", "ubuntu1.ir", "www.ubuntu1.ir"]'

[vm2]
192.168.55.101 ansible_user=rocky1 server_name='192.168.55.101 rocky.ir www.rocky.ir' DNS_Address='nameserver 10.202.10.10' ALLOWED_HOSTS='ALLOWED_HOSTS=["192.168.55.101", "rocky.ir", "www.rocky.ir"]'

[vm3]
192.168.66.101 ansible_user=ubuntu2 server_name='192.168.66.101 ubuntu2.ir  www.ubuntu2.ir' DNS_Address='nameserver 10.202.10.10' ALLOWED_HOSTS='ALLOWED_HOSTS=["192.168.66.101", "ubuntu2.ir", "www.ubuntu2.ir"]'
```

### Playbook
A playbook.yaml file in Ansible defines a set of tasks for automating configurations, deployments, and management of systems. Each playbook consists of one or more "plays," which map hosts to tasks, specifying what actions to perform on which servers.

```
- hosts: router
  become: true
  tasks:

    - name: NAT Masquerading
      ansible.builtin.command:
        cmd: iptables -t nat -A POSTROUTING -o {{ bridgedInterface  }} -j MASQUERADE



- hosts: all
  become: true
  tasks:

    - name: Add  DNS servers
      lineinfile:
        path: /etc/resolv.conf
        regexp: '^nameserver.*'
        line: "{{ DNS_Address  }}"
        dest: /etc/resolv.conf

    - name: Update and upgrade packages on Ubuntu
      ansible.builtin.apt:
        update_cache: yes
        upgrade: yes
      when: ansible_distribution == "Ubuntu"

    - name: Update and upgrade packages on CentOS/Rocky Linux (Yum/DNF)
      ansible.builtin.yum:
        name: "*"
        state: latest
      when: ansible_distribution in ["CentOS", "Rocky"]



- hosts: all
  become: true
  tasks:

    - name: Install Nginx on Ubuntu
      ansible.builtin.apt:
        name: nginx
        state: latest
      when: ansible_distribution == "Ubuntu"

    - name: Install Nginx on CentOS/Rocky Linux
      ansible.builtin.yum:
        name: nginx
        state: latest
      when: ansible_distribution in ["CentOS", "Rocky"]

    - name: Install pip and Python packages on Ubuntu
      ansible.builtin.apt:
        name:
          - python3
          - python3-pip
          - python3-virtualenv
          - python3-venv
          - libaugeas0
        state: latest
      when: ansible_distribution == "Ubuntu"

    - name: Install EPEL and Remi repository
      ansible.builtin.yum:
        name:
          - epel-release
          - yum-utils
        state: present
      when: ansible_distribution in ["CentOS", "Rocky"]

    - name: Install pip and Python packages on CentOS/Rocky Linux
      ansible.builtin.yum:
        name:
          - python3
          - python3-pip
          #- python3-virtualenv
          #- python3-venv
          - augeas-libs
        state: latest
      when: ansible_distribution in ["CentOS", "Rocky"]



- hosts: django
  become: true
  tasks:

    - name: Create Directory
      file:
        path: ~/mysite
        state: directory

    - name: Create virtualenv  & Install django in it
      pip:
        name: django
        version: 5.1.1
        virtualenv: ~/mysite/venv
        virtualenv_command: virtualenv

    - name: Create django project
      command: "~/mysite/venv/bin/django-admin startproject djangoProject chdir=~/mysite/"

    - name: Set ALLOWED_HOSTS in Django settings.py
      ansible.builtin.lineinfile:
        path: ~/mysite/djangoProject/djangoProject/settings.py
        regexp: '^ALLOWED_HOSTS\s*=\s*\[.*\]'
        line: "{{ ALLOWED_HOSTS  }}"

    - name: Change Django urls.py
      ansible.builtin.lineinfile:
        path: ~/mysite/djangoProject/djangoProject/urls.py
        regexp: 'admin/'
        line: 'path("django/", admin.site.urls),'

    - name: Change static root in Django settings.py
      ansible.builtin.lineinfile:
        path: ~/mysite/djangoProject/djangoProject/settings.py
        regexp: 'STATIC_URL'
        line: "STATIC_URL = 'django/static/'"

    - name: Remove Nginx default file in sites-available
      ansible.builtin.file:
        path: "/etc/nginx/sites-available/default"
        state: absent

    - name: Remove Nginx default file in sites-enabled
      ansible.builtin.file:
        path: "/etc/nginx/sites-enabled/default"
        state: absent

    - name: Copy Nginx default file from template
      ansible.builtin.template:
        src: ./templates/default.conf.j2
        dest: /etc/nginx/conf.d/default.conf

    - name: Restart Nginx service
      command: "systemctl restart nginx"

    - name: Migrate django
      command: "~/mysite/venv/bin/python3 ~/mysite/djangoProject/manage.py migrate"

    - name: Run Django development server
      ansible.builtin.shell: "~/mysite/venv/bin/python3 manage.py runserver 0.0.0.0:8000"
      args:
        chdir: "~/mysite/djangoProject"
      async: 3600  
      poll: 0 



- hosts: laravel
  become: true
  tasks:


    - name: Install php and extentions on ubuntu
      ansible.builtin.apt:
        name:
          - php
          - php-cli
          - php-mbstring
          - php-xml
          - php-sqlite3
          - unzip
          - curl
        state: present
      when: ansible_distribution == "Ubuntu"

    - name: Install Remi repository for CentOS 8/Rocky Linux 8
      dnf:
        name: https://rpms.remirepo.net/enterprise/remi-release-8.rpm
        state: present
      when: ansible_distribution == "['CentOS', 'Rocky']"

    - name: Enable PHP 8.0 Remi module
      command: dnf module enable php:remi-8.0 -y
      ignore_errors: true
      when: ansible_distribution == "['CentOS', 'Rocky']"
    
    - name: Install PHP and required extensions
      ansible.builtin.yum:
        name:
          - php
          - php-cli
          - php-fpm
          - php-mysqlnd
          - php-xml
          - php-json
          - php-mbstring
          - php-zip
          - php-gd
          - php-curl
        state: latest
      when: ansible_distribution in ["CentOS", "Rocky"]

    - name: Download Composer installer
      get_url:
        url: https://getcomposer.org/installer
        dest: /tmp/composer-setup.php

    - name: Install Composer
      command: "php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer"

    - name: Executable composer
      file:
        path: /usr/local/bin/composer
        mode: '0755'

    - name: Create Laravel project
      command: "/usr/local/bin/composer create-project laravel/laravel ~/laravelProject --no-interaction"

    - name: Run Laravel development server
      ansible.builtin.shell: "php artisan serve --host 0.0.0.0 --port 8001"
      args:
        chdir: "~/laravelProject"  
      async: 3600  
      poll: 0  

    - name: Change Laravel web.php route
      ansible.builtin.lineinfile:
        path: ~/laravelProject/routes/web.php
        regexp: 'Route::get'
        line: 'Route::get("/laravel", function () {'

    - name: Copy Nginx default file from template
      ansible.builtin.template:
        src: ./templates/default.conf.j2
        dest: /etc/nginx/conf.d/default.conf

    - name: Restart Nginx service
      command: "systemctl restart nginx"

    - name: Add HTTP service to the public zone
      firewalld:
        zone: public
        service: http
        permanent: yes
        state: enabled
      when: ansible_distribution in ["CentOS", "Rocky"] 

    - name: Reload firewalld
      firewalld:
        state: reloaded
      when: ansible_distribution in ["CentOS", "Rocky"]

    - name: command
      command: "setsebool -P httpd_can_network_connect 1"
      when: ansible_distribution in ["CentOS", "Rocky"]
```

#### Source files in git:
```
https://github.com/JavadSafari81/ansible-vms.git
```
