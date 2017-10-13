# Ansible Introduction

Dong Bin

---
# Background
- Painful to ssh machines
- Bash script is ugly even for basic tasks
- Need a simple way to manage many groups of servers

---
# Bash Programming is Painful
- Not idempotent
- Environment dependent
- No abstraction
```bash
#!/bin/sh
set -e
filelist=$(find . -type f)
while [ "$filelist" != "" ]
do
  line=$(echo "$filelist" | head -1)
  filelist=$(echo "$filelist" | tail +2)
  ls "$line"
  cat "$line"
done
```
---
# Install
```Bash
brew install python
pip install ansible
```
```
# /etc/ansible/hosts
[dispatch]
g1-wx-kf-v04
g1-wx-kf-v05

[sale-bg]
g1-wx-kf-v02
g1-wx-kf-v03
```
---
# Adhoc Usage
```bash
ansible all -a '/bin/echo hi'
ansible dispatch -m shell -a 'ps -aux|grep grpc'
```
---

# Ansible Modules
```bash
ansible all -m yum -a 'name=nginx state=installed'
ansible all -m service -a 'name=nginx state=started'

ansible all -m copy -a 'src=foo dest=/tmp/bar mode=0644'

ansible dispatch -m replace -a 'path=/etc/init/smart-dispatch-grpc.conf regexp=5001 replace=6001'  -b
ansible dispatch -m replace -a 'path=/etc/init/smart-dispatch-grpc.conf regexp=grpc.new.log replace=grp

```
---
# Playbook
- YAML files
- Declaratively
- Reuse
```
ansible-playbook playbook.yml
```
---
# Example
```yaml
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: name=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running (and enable it at boot)
    service: name=httpd state=started enabled=yes
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```
---
# Q&A
