# s9s-ansible
Examples of Ansible snippets for s9s CLI

# How To Execute Playbooks
Every playbook we have points a specific environment. You need to specify the type of environment, for example if it's prod, qa, or whatever it is as long as it exist as a directory in inventory/group_vars/{{environ}} directory.

For example, since we have a sample inventory/group_vars/prod, then you can have environ=prod just like below:
```
cd /the/path/where/s9s-ansible/
root@pupnode6:/vagrant/non-ansible-files/s9s_ansible# ansible-playbook -i inventory/prod -v playbooks/add-loadbalancer-postgresql-pgbouncer.yml -e "environ=prod"
```


For MySQL/MariaDB/Percona Replication, run as,
```
cd /the/path/where/s9s-ansible/
root@pupnode6:/vagrant/non-ansible-files/s9s_ansible# ansible-playbook -i inventory/prod -v playbooks/add-loadbalancer-postgresql-pgbouncer.yml -e "environ=prod" -e "vendor=<vendor>"
```
where `vendor` are `mysql`,`percona`, or `mariadb`.
