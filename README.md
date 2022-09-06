# s9s-ansible
The **s9s-ansible** are ansible scripts consisting of playbooks that takes action for the ClusterControl instead of manually following the procedures based on the UI, **s9s-ansible** will do the work by automating the job. Thus, removing the need to do it manually, but instead relies on an efficient automation and quick modification of values based on the given parameters giving you the benefit to quickly do the job or determine the outcome of the job quickly. The job activity is based on the playbook that is chosen to be executed, whereas the playbook depends on the given paramater variables defined in each of its inventory files given.

Presently, the **s9s-ansible** playbooks applies the following job activities and actions :
| Actions        | Job Description |
| ------------- |:-------------|
| Database cluster deployment     | Deploy these types of databases: MySQL/MariaDB/Percona/Galera, PostgreSQL, MongoDB, Redis, SQL Server (Availability Groups), Elasticssearch |
|  Removal of database cluster     | Delete any cluster in a given inventory     |
| Database Proxy Deployment | Deploy ProxySQL, HAProxy, PgBouncer, Keepalived  |
| Removal of Proxies | Ability to remove database proxies/load balancers in a given inventory |
| Enable/Disable Dashboards (metrics collector) | Will install (enable) Prometheus and its exporters, or uninstall (disable) Prometheus and its exporters |
| Enable/Disable query monitor (agents for applidable db types) | Will install (enable) query monitor agents, or uninstall (disable) query monitor agents |
| Enable/Disable WAL Archiving | Enables WAL archiving or disables WAL archiving. This is only applicable for PostgreSQL cluster |
| Enable/Disable Audit plugin | Enables the audit plugin or disables the audit plugin. This is only applicable currently for a MariaDB Cluster deployment|
| Add replication slave | Ability to add a replica for an existing replication cluster (MySQL/MariaDB/Percona/PostgreSQL) |
| Enable/Disable Auto Recovery | This allows to enable or disable (turn ON/OFF modes) auto recovery |
| Schedule Maintenance Window | Allows to create a scheduled maintenance window in an entire cluster or on a node basis |
| Promoe Slave/Standby | Promotes a replica or standby to a role of a primary |
| Take backup | Ability to create a backup on the fly |
| Create/Remove backup schedule | Ability to create ~~or remove a backup schedule~~ |
| Restore from backup | Ability to restore from a backup |


# Getting started with s9s-ansible?
Ansible is known to operate as a tool for automation especially in DevOps, or CI/CD systems. However, the **s9s-ansible** uses ansible as the wrapper to build up this project. However, the main heart and soul that does the job is in fact the tools made by Severalnines which is the [s9s-cli](https://github.com/severalnines/s9s-tools) tool and also some integration with [RCP v2](https://severalnines.com/downloads/cmon/cmon-docs/current/rpcv2.html).

Since **s9s-ansible** uses the s9s-tools and some RPC v2 (uses cURL command to invoke the API via given URL), **s9s-ansible** requires only to be run under where the ClusterControl host is installed and thus, it connects via 127.0.0.1 or localhost. Ideally, it can work using TCP as long as a valid FQDN, hostname, or IP address is specified. Although, it has never been tested and is never intended yet to be supported initially. Currently, the ideal usage of **s9s-ansible** is to run in the ClusterControl controller host, and ansible has to be installed on the same host as well, where ClusterControl is hosted and running.

## Using s9s-ansible
In order to use the **s9s-ansible**, you must download or clone the latest or main brach of this project.

E.g.
```
$ git clone git@github.com:severalnines/s9s-ansible.git
```

You can store it in any location of your Linux environment. Preferrably, the **s9s-ansible** is only tested and worked on a Linux environment. Other operating systems other than Linux is not supported.

Once you have the package inside your system, go to the directory of your **s9s-ansible** copy. copy the ```.sample-ansible.cfg``` and rename it as ```ansible.cfg```.

Basically, the ansible.cfg as of this writing only containts the definition of your inventory paramter and where it is located.

E.g.
```
geekgogie@ubuntu:/opt/devops/s9s-ansible$ cat ansible.cfg
[defaults]
inventory=inv/sample_prod
```

For those who are already familiar with Ansible especially advance users, you might have already known what's the purpose of this parameter definition in the ansible.cfg config file.

For those who are not familar, the inventory parameter points to the ```inv/sample_prod``` file. The ```inv/sample_prod``` file containts only the following,
```
$ cat inv/sample_prod
[defaults]
environ=sample_prod

[sample_prod]
127.0.0.1
```
One thing that is very important here. The name _sample_prod_ is only an arbitrary tag name that you would like to name in your environment. Supposedly, you have a *production*, *qa*, *dev*, *staging*, or whatever naming conventions you call it such as domain-name basis, you can replace the _sample_prod_ to this name. In that case, let say you have aws_prod, and aws_qa, then the file ```inv/sample_prod``` can be set to ```inv/aws_prod``` or ```inv/aws_qa```. Then do not forget also to replace the contents of your inv/sample_prod file to have,

```
[defaults]
environ=aws_qa

[aws_qa]
127.0.0.1
```

Apart from these files, the **s9s-ansible** contains the inventory directory where it groups the type of environment and is synced also to the type of the environment you defined (for example) in file ```inv/sample_prod```, it has
```
$ tree -L 1 inventory/group_vars/
inventory/group_vars/
├── sample_prod
└── sample_qa

2 directories, 0 files
```

If you have inv/aws_qa for example, then create a directory named ```inventory/group_vars/aws_qa```. This is where your inventories for your QA environment will be located.

Lastly, the **s9s-ansible** selects a common inventory file where you can add the general parameters, if you need to be defined. For example, your operating system user, the ssh location keys. These variables are commonly uniform in all the database nodes since it is required by ClusterControl to have the user eixst or to have it all the same for all your database nodes. See [here](https://docs.severalnines.com/docs/clustercontrol/requirements/operating-system-user/) for more info. Now, this common file is located in ```inventory/group_vars/sample_prod/sample_prod_common```. In our given example, given that you have created ```inventory/group_vars/aws_qa``` directory, do not forget to create a file i.e. ```inventory/group_vars/aws_qa/aws_qa_common```. You might likely want to copy the contents of the sample file  ```inventory/group_vars/sample_prod/sample_prod_common``` and rename it accordingly. 


For those who are familiar with Ansible, the ```inventory``` parameter found in the ```ansible.cfg``` just simplifies the command where you don't need anymore to provide the ```-i <location-of-inventory-file>``` option when running ```ansible-playbooks``` command.


Now that it's already setup, you are now ready to setup and run a playbook. Let's proceed on the next section.



## Running your ansible-playbook
The **s9s-ansible** places the playbooks inside the playbook directory. Currently, this is how the **s9s-ansible** playbooks directory looks like:
```
$ ~/s9s-ansible$ tree --dirsfirst playbooks/
playbooks/
├── helpers
│   ├── multi-node-deploy-haproxy.yml
│   ├── multi-node-deploy-proxysql.yml
│   ├── multi-node-remove-haproxy.yml
│   ├── multi-node-remove-proxysql.yml
│   ├── multi-node-schedule-bynode-maintenance.yml
│   ├── multi-node-schedule-delbynode-maintenance.yml
│   ├── multi-node-schedule-delcluster-maintenance.yml
│   ├── multi-node-schedule-pernode-maintenance.yml
│   └── multi-node-unregister-node.yml
├── add-loadbalancer-haproxy.yml
├── add-loadbalancer-keepalived.yml
├── add-loadbalancer-postgresql-pgbouncer.yml
├── add-loadbalancer-proxysql.yml
├── add-loadbalancer-replication-proxysql.yml
├── add-node-mysql-replication.yml
├── backup-create-ondemand.yml
├── backup-create-scheduled.yml
├── backup-restore.yml
├── deploy-elasticsearch.yml
├── deploy-galera-cluster.yml
├── deploy-mariadb-galera.yml
├── deploy-mariadb-replication.yml
├── deploy-mongodb-rs.yml
├── deploy-mssql.yml
├── deploy-mysql-replication.yml
├── deploy-percona-galera.yml
├── deploy-percona-replication.yml
├── deploy-postgresql.yml
├── deploy-redis.yml
├── drop-cluster.yml
├── manage-audit-logging.yml
├── manage-auto-recovery.yml
├── manage-dashboard-agents.yml
├── manage-promote-replica.yml
├── manage-query-monitor.yml
├── manage-schedule-maintenance.yml
├── manage-wal-archiving.yml
└── unregister-node.yml

1 directory, 38 files
```
where all the main playbooks are located in ```playbooks/``` directory, whereas the helpers are located in ```playbooks/helpers``` directory. The s9s-ansible playbooks cannot run on itself, it depends also in the inventories. The inventories is where the user shall modify and designate the specific values to be assigned such as cluster name, hostname/ip addresses, username/password, config location, etc. In the example inventory, you'll see that these are organize into a particular directory,
```
root@pupnode6:/vagrant/files/s9s-ansible# tree -L 1 -d inventory/group_vars/sample_prod/
inventory/group_vars/sample_prod/
├── add_lb
├── backup
├── deploy
├── drop_cluster
├── manage
└── unregister

6 directories
```
The directories above explains the following:
- *add_lb*. The directory for inventory files where adding of load balancers or load balancer related shall be located or placed.
- *backup*. The directory for inventory files where adding of operating and managing backup related shall be located or placed.
- *deploy*. The directory for inventory files for deploying your database clusters
- *drop_cluster*. The directory for inventory files related to dropping a cluster
- *manage*. The directory for inventory files for managing your database such as enabling dashboard agents, enable auto recovery, scheduling maintenance window, etc.
- *unregister*. - is where the directory for inventory files related to unregistering a node(s)


In this section, we will do the following actions:
- deploy a MariaDB Replication Cluster
- deploy a ProxySQL load balancer to this cluster
- deploy a HAProxy load balancer to have HA for the ProxySQL nodes
- create a backup for this cluster
- restore a backup for this cluster
- create a scheduled backup for this cluster
- deploy dashboard or enable prometheus and its agents
- deploy query monitoring agents
- uninstall HAProxy and ProxySQL
- uninstall a database node (replica)
- create a replica
- drop the cluster

Before going through the actions, here are the sample prerequisite configs we have set in this example.
```
root@pupnode6:/vagrant/files/s9s-ansible# cat ansible.cfg
[defaults]
inventory=inv/staging
root@pupnode6:/vagrant/files/s9s-ansible# cat inv/staging
[defaults]
environ=staging

[staging]
127.0.0.1
root@pupnode6:/vagrant/files/s9s-ansible# cat inventory/group_vars/staging/staging_common
os_user: "vagrant"
os_key_file: "/home/vagrant/.ssh/id_rsa"
```


## When running a playbook
The **s9s-ansible** playbooks requires certain variables to be defined when running ansible-playbook. Generally, the most required variable is ```environ``` and ```deploy_id```. The variable ```environ``` poins to the type of environment. The value must coincide or sync to the assigned value you have in inv/staging (in the example above). So in the example above, the ```environ == staging```. The ```deploy_id``` variable is an inventory file itself that serve as like a token identity in order to make the ansible playbook deployment working. In that case the ```deploy_id``` value is a file that points to the designated directory based on its desired actions.


###  Deploy a MariaDB Replication Cluster
To deploy a MariaDB Replication cluster, use the playbook ```deploy-mariadb-replication.yml```. Since this is a deployment, an inventory shall be created under ```inventory/group_vars/staging/deploy/```. Let's create an inventory file called *mariadb_repl_c1* with the following contents:
```
root@pupnode6:/vagrant/non-ansible-files/s9s-ansible# cat inventory/group_vars/staging/deploy/mariadb_repl_c1
cluster_name: "mariadb-replication-c1"
cluster_type: replication
vendor: "mariadb"
version: "10.5"
db_admin_user: root
db_admin_password: mydbP@55w0rd
db_nodes:
    - host: 192.168.40.58
      port: 3306
      node_type: master
    - host: 192.168.40.59
      port: 3306
      node_type: slave
# cluster_tags should be comma delimited without any space in between
cluster_tags: "prod,c1-group"
wait_log: "--wait --log"
prefix: "mysql://"
```
If you have noticed, the ```cluster_name``` value resembles to the filename of the inventory, i.e. ```inventory/group_vars/staging/deploy/mariadb_repl_c1```. We suggest that when creating the inventory which represents the ```deploy_id``` value in the playbook, make sure to name it accordingly that is easy for you to identify, such as your ```cluster_name``` value.

Running the command below, shall deploy the MySQL Replication cluster.
```
$ ansible-playbook playbooks/deploy-mysql-replication.yml -e "environ=staging deploy_id=mariadb_repl_c1" -v
```

### Deploy a ProxySQL load balancer to this cluster
Make sure you have the ```add_lb``` directory in your inventory. In this example, I have the following directory created ``` inventory/group_vars/staging/add_lb/```. Now inside this directory, let's create the inventory file which will serve as the ```deploy_id``` value in the playbook. In this example, I named the file as _mariadb_repl_proxysql_c1_ so I can easily identify it to what cluster and to what type of load balancer it does add or deploy. Below is the following contents:
```
$ cat inventory/group_vars/staging/add_lb/mariadb_repl_proxysql_c1
cluster_name: "mariadb-replication-c1"
cluster_type: replication
proxy_admin_user: "proxysql-admin"
proxy_admin_password: "admin"
proxy_monitor_user: "proxysql-monitor"
proxy_monitor_password: "monitor"
prefix: "proxysql://"
proxy_hosts:
    - host: 192.168.40.58
      port: 6032
    - host: 192.168.40.59
      port: 6032
wait_log: "--wait --log"

## optional db user to add
db_username: "proxysql-paul"
db_password: "admin"
db_database: "*.*"
db_privs: "ALL PRIVILEGES"
```
To deploy a load balancer to the cluster that was deployed or created previously, run the following playbook:
```
$ ansible-playbook playbooks/add-loadbalancer-proxysql.yml -e "environ=staging deploy_id=mariadb_repl_proxysql_c1" -v
```
In this playbook, the add-loadbalancer-proxysql.yml playbook allows multiple node installation of ProxySQL which makes it effortless and efficient deployment of ProxySQL.

### Deploying HAProxy load balancer for the MySQL nodes
Using **s9s-ansible** for deploying HAProxy as your chosen load balancer supports multi-deployment feature. The multi-deployment has to be setup one at a time but it still follows the sequential approach that ClusterControl deploys in the UI. What that means is that, since the inventory shall be defined in a per-cluster basis, therefore, the multi-deployment is taken sequentially which takes one job at a time and the following jobs shall be put into pending state until the current one finishes and proceeds on the next sequence in the job list.

In this example, we have already deployed the MariaDB Replication cluster, this can be verified by running the ```s9s cluster --list --long``` command. For example,
```
root@pupnode6:/vagrant/non-ansible-files/s9s-ansible# s9s cluster --list --long
ID STATE   TYPE              OWNER    GROUP  NAME                   COMMENT
79 STARTED mongodb           test_dba admins mongodb-mdbgrp         All nodes are operational.
82 STARTED replication       test_dba admins mysql-repl5.7          All nodes are operational.
83 STARTED postgresql_single test_dba admins postgres-c1            All nodes are operational.
92 STARTED replication       test_dba admins mariadb-replication-c1 All nodes are operational.
Total: 4
```
Wherein, the deployed cluster is named as *mariadb-replication-c1* in this example.

To deploy HAProxy on its target database nodes, you just need the ```playbooks/add-loadbalancer-haproxy.yml``` playbook. Before we run this, we need to create the inventory file ```mariadb_repl_haproxy_c1``` and we have to place this in ```inventory/group_vars/staging/add_lb/``` directory. If the directory does not yet exist, please create it first as you have to make sure this is located in directory ```*/add_lb```.

In your inventory file, we have the following params:
```
## RPC V2 compatible
use_rpcv2: true
cluster_name: "mariadb-replication-c1"
cluster_type: "replication"
proxy_type: "haproxy"
prefix: "haproxy://"
proxy_hosts:
    - host: 192.168.40.58
      ha_port: 9600
      rw_splitting: true
      node_type: active
      backend_name_ro: haproxy_2208_ro
      backend_name_rw: haproxy_2207_rw
      node_addresses: 192.168.40.58:3306:active,192.168.40.59:3306:backup
      lb_admin: myhaproxyAdmin
      lb_password: myhaproxyPassword
      disable_firewall: true
      disable_selinux: true
      overwrite_mysqlchk: true
      lb_policy: leastconn
      max_connection_be: 64
      max_connection_fe: 8192
      ro_port: 3308
      rw_port: 3307
      rw_splitting: true
      stats_socket: /var/run/haproxy.socket
      timeout_client: 10800
      timeout_server: 10800
      xinetd_allow_from: 0.0.0.0/0
      build_from_source: false	  
    - host: 192.168.40.59
      ha_port: 9600
      rw_splitting: true
      node_type: active
      backend_name_ro: haproxy_2208_ro
      backend_name_rw: haproxy_2207_rw
      node_addresses: 192.168.40.58:3306:active,192.168.40.59:3306:backup
      lb_admin: myhaproxyAdmin
      lb_password: myhaproxyPassword
      disable_firewall: true
      disable_selinux: true
      overwrite_mysqlchk: true
      lb_policy: leastconn
      max_connection_be: 64
      max_connection_fe: 8192
      port: 9600
      ro_port: 3308
      rw_port: 3307
      rw_splitting: true
      stats_socket: /var/run/haproxy.socket
      timeout_client: 10800
      timeout_server: 10800
      xinetd_allow_from: 0.0.0.0/0
      build_from_source: false
rpc_user: myuser
rpc_password: myuserpass
# RPC V2 URL should have the v2/jobs as it's sub-directory in the URL. If you need to change the URL, likely you'll change only the IP address 
## such as your hostname or any FQDN that points to the RPC V2 running in your CC Controller host. In this example, we're using localhost.
rpc_v2_url: https://127.0.0.1:9501/v2/jobs
```
This shall use the RPC V2 and shall feed the designated values to the RPC using cURL. Replace the following params to your desired value. Once you are ready, then invoke the following playbook:
```
$ ansible-playbook playbooks/add-loadbalancer-haproxy.yml -e "environ=staging deploy_id=mariadb_repl_haproxy_c1" -v
```

### Create a backup for this cluster
### Restore a backup for this cluster
### Create a scheduled backup for this cluster
### Deploy dashboard or enable prometheus and its agents
### Deploy query monitoring agents
### Uninstall HAProxy and ProxySQL
### Uninstall a database node (replica)
### Create a replica
### Drop the cluster




