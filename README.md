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
│   ├── multi-node-deploy-haproxy_rpcv2.yml
│   ├── multi-node-deploy-proxysql.yml
│   ├── multi-node-remove-haproxy.yml
│   ├── multi-node-remove-proxysql.yml
│   ├── multi-node-schedule-bynode-maintenance.yml
│   ├── multi-node-schedule-delbynode-maintenance.yml
│   ├── multi-node-schedule-delcluster-maintenance.yml
│   ├── multi-node-schedule-pernode-maintenance.yml
│   └── multi-node-unregister-node.yml
├── add-database-node.yml
├── add-loadbalancer-haproxy.yml
├── add-loadbalancer-keepalived.yml
├── add-loadbalancer-postgresql-pgbouncer.yml
├── add-loadbalancer-proxysql.yml
├── backup-create-ondemand.yml
├── backup-create-scheduled.yml
├── backup-restore.yml
├── deploy-elasticsearch.yml
├── deploy-galera-cluster.yml
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

1 directory, 36 files
```
where all the main playbooks are located in ```playbooks/``` directory, whereas the helpers are located in ```playbooks/helpers``` directory. The s9s-ansible playbooks cannot run on itself, it depends also in the inventories. The inventories is where the user shall modify and designate the specific values to be assigned such as cluster name, hostname/ip addresses, username/password, config location, etc. In the example inventory, you'll see that these are organize into a particular directory,
```
root@pupnode6:/vagrant/files/s9s-ansible# tree -L 1 -d inventory/group_vars/sample_prod/
inventory/group_vars/sample_prod/
├── add_lb
├── add_node
├── backup
├── deploy
├── drop_cluster
├── manage
└── unregister

7 directories
```
The directories above explains the following:
- *add_lb*. The directory for inventory files where adding of load balancers or load balancer related shall be located or placed.
- *add_node*. The directory for inventory files where adding of nodes to existing clusters in the ClusterControl.
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
- unregister HAProxy, ProxySQL, and database node (replica)
- create or add a replica
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

To deploy HAProxy on its target database nodes, you just need the ```playbooks/add-loadbalancer-haproxy.yml``` playbook. Before we run this, we need to create the inventory file ```mariadb_repl_haproxy_c1``` and we have to place this in ```inventory/group_vars/staging/add_lb/``` directory. If the directory does not yet exist, please create it first as you have to make sure this is located in directory ```*/add_lb```. Take note that the directories where the inventories are placed are organized based on its type of job. Since this HAProxy is treated as load balancer, then this is shall be placed in `add_lb` directory.

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
Backup management is one of the primary jobs that can be done using **s9s-ansible** scripts. In this section, we will deploy a backup using on-demand which takes the backup on the fly and is equivalent to do the on-demand capability if you have to do this action using the ClusterControl web UI.

As we have shown earlier, we have the cluster named `mariadb-replication-c1` deployed. This will be our target cluster. Since the job is about backup, then we need to create the directory ```inventory/group_vars/staging/backup/```.

To create the backup on-demand, we need to create the inventory and we'll name the file as `mariadb_repl_c1_xtrabackupfull` and save it to `inventory/group_vars/staging/backup/` directory. The file shall have the params and its values as shown below:

```
# cat inventory/group_vars/staging/backup/mariadb_repl_c1_mysqldump
cluster_name: "mariadb-replication-c1"
cluster_type: replication
backup_method: "mysqldump"
backup_directory: /home/vagrant/backups
store_on_controller: false
backup_retention: 2
node:
    - host: 192.168.40.59
      port: 3306
```

The inventory file itself uses mysqldump as the choosen backup method and shall have not store the backup in the controller but on the node itself. `backup_retention` param sets 2 days of retention so after threshold is lapse, backup shall be deleted. The target host is the replica so specifying the node's host and port is required.

To create and perform the backup on-demand, simply run the following playbook as follows:
```
$ ansible-playbook playbooks/backup-create-ondemand.yml -e "environ=staging deploy_id=mariadb_repl_c1_mysqldump" -v
```

### Create a scheduled backup for this cluster
Creating a scheduled backup is just the same as running the backup on-demand but with slight differences. The inventory file needs to specify the schedule of the backup. In this case, the parameter named `backup_schedule` takes the value of a cron-tab scheduling [syntax format](https://www.seobility.net/en/wiki/CronJob#Structure_and_syntax_of_a_CronTab_file).

To create a scheduled backup, create an inventory file, and by following our desired naming convention, let's call it `mariadb_repl_c1_mariackupfull`. This file name means it belongs to the MariaDB Replication cluster where `c1` is one of the tags or an arbitrary name for a data center, concatenated with `mariabackupfull` as the desired backup method to be used. 
```
# cat inventory/group_vars/staging/backup/mariadb_repl_c1_xtrabackupfull
cluster_name: "mariadb-replication-c1"
cluster_type: replication
backup_method: "mariabackupfull"
backup_directory: /home/vagrant/backups
store_on_controller: true
backup_schedule: 10 22 * * *
backup_retention: 2
node:
    - host: 192.168.40.59
      port: 3306
wait_log: '--wait --log'
```
This means that it will store on controller host, backup schedule is set to run every 22:10 with a retention of 2 days targetting host 192.168.40.59 (a replica) where the MariaDB node runs on port 3306.

To execute the playbook, we'll call and run playbook `backup-create-scheduled.yml`. In this example, we'll run the command below:
```
$ ansible-playbook playbooks/backup-create-scheduled.yml  -e "environ=staging deploy_id=mariadb_repl_c1_xtrabackupfull" -v
```
### Restore a backup for this cluster
Restoring a backup for this cluster is dependent on the backups that were made using ClusterControl. Either the backup was taken using **s9s tools**, through the ClusterControl web UI, or using this **s9s-ansible** scripts. Backups taken using ClusterControl are assigned with unique backup title, which means, restoring a backup using **s9s-ansible** requires that you should have the knowledge of the backup you are aiming to restore and to what node (target node). The node has to be part of the cluster for this backup restore job to be successful.

To determine the list of backup with its backup title, simply run the `s9s backup` command as shown below:
```
$ s9s backup --list --long --cluster-id 92
ID  PI CID V I STATE     OWNER    HOSTNAME      CREATED  SIZE      TITLE
143  -  92 - F COMPLETED test_dba 192.168.40.59 01:13:22  51595297 BACKUP-143
146  -  92 - F COMPLETED test_dba 192.168.40.59 22:10:00 132292889 BACKUP-146
Total 2
```
In this example, the cluster id is 92 for which this list of backups are taken from and as the list shows, the source are coming from one origin, i.e. 192.168.40.59 (serves as a replica for this cluster). Now, we have to backup titles namely BACKUP-143 and BACKUP-146. We'll try to restore *BACKUP-146* as its latest backup to the primary/master host for this cluster, i.e. 192.168.40.58.

Before proceeding, we need to create the inventory file first. Create a file named `mariadb_repl_c1_restore` and store it in `inventory/group_vars/staging/backup/` directory since this action is related to backups. The params contained for this file is simple as shown below:
```
$ cat inventory/group_vars/staging/backup/mariadb_repl_c1_restore
cluster_name: "mariadb-replication-c1"
cluster_type: replication
backup_title: BACKUP-146
node:
    - host: 192.168.40.58
      port: 3306
wait_log: '--wait --log
```
which means that, the `backup_title` we have assigned is *BACKUP-146* and targetting host *192.168.40.58:3306* specifically.

To run the script for backup restore, simply run the playbook `backup-restore.yml` as shown below:
```
$ ansible-playbook playbooks/backup-restore.yml -e "environ=staging deploy_id=mariadb_repl_c1_restore" -v
```

### Enabling Dashboard (enable prometheus and its agents)
Deploying the dashboard using **s9s-ansible** uses RPC V2 as there's no way to use **s9s-tools** install (only supported by **s9s-tools**) and uninstalling the dashboard. In this section, we'll use the deployment of dashboard. This management job allows not only to enable (deploy) or disable (uninstall) the dashboards. With installing, it will deploy all the agents to the database nodes or load balancers that are supported by ClusterControl. Uninstalling also will uninstall the agents as well as the dashboard, which install/deploys Prometheus, and unregisters it to the Prometheus.

Since this can be done using **s9s-ansible** using RPC V2 by simply invoking cURL, that means `--wait --log` parameter is not supported in this playbook.

First, we need to create the directory `inventory/group_vars/staging/manage` since this job is part of managing the agents (enable/disable). Then, create the inventory file and let's call it as `dashboard_mariadb_repl_c1` and save it to directory `inventory/group_vars/staging/manage`. Use this parameter contents below but make sure you modify the values according to your environment.
```
$ cat inventory/group_vars/staging/manage/dashboard_mariadb_repl_c1
cluster_name: "mariadb-replication-c1"
cluster_type: replication
enable_dash: "{{ enable |d(true) }}"
let_it_run_and_no_uninstall: false
data_retention: 15d
data_retention_size: 0
datadir:
prometheus_host: 192.168.40.63
scrape_interval: 10s
# make sure when adding values to the parameters make sure you have escaped the quotes and the values are inside
## the quotes as well.
configuration: '[
{
  \"arguments\": \"\",
  \"job\": \"node\",
  \"scrape_interval\":\"\"
},
{
  \"arguments\": \"--collect.perf_schema.eventswaits --collect.perf_schema.file_events --collect.perf_schema.file_instances --collect.perf_schema.indexiowaits --collect.perf_schema.tableiowaits --collect.perf_schema.tablelocks --collect.info_schema.tablestats --collect.info_schema.processlist --collect.binlog_size --collect.global_status --collect.global_variables --collect.info_schema.innodb_metrics --collect.slave_status --collect.info_schema.userstats\",
  \"job\": \"mariadbd\",
  \"scrape_interval\":\"\"
}
]'
rpc_user: myuser
rpc_password: myuserpass
rpc_v2_url: https://127.0.0.1:9501/v2/jobs
```
In the params above, what's most important you can modify is the `prometheus_host`, `data_retention`, `data_retention_size`, `rpc_user`, `rpc_password`. The `configuration` is also very important if you need to add custom arguments. The `rpc_v2_url` maybe constant but in case we support hostname/IP aside from using localhost or 127.0.0.1, then you can change this as well in the future. The variable `let_it_run_and_no_uninstall` is important only during uninstalling or disabling of the dashboard. Meaning, if you set this to _true_, it will disable Prometheus and agents but allow it to run, otherwise if _false_, then it will totally remove and uninstall Prometheus and its agents/exporters. The `configuration[0].arguments.job` and `configuration[1].arguments.job` are set as values of `node` and the second `job` value depends on what type of cluster it is and its binary command to be invoked. For example, since we are using MariaDB, it uses binary `mariadbd`. For MySQL/Percona Server, it uses `mysqld`. For MongoDB, it uses `mongod`. For PostgreSQL, it uses `postgres`.

The `rpc_user` and `rpc_password` are the user/password combination you have created to execute and run **s9s-tools** commands.

After all params i.e. key/value pairs are set, now we're ready to execute the playbook `/manage-dashboard-agents.yml` and run the following command as follows:
```
$ ansible-playbook playbooks/manage-dashboard-agents.yml  -e "environ=staging deploy_id=dashboard_mariadb_repl_c1 enable=true" -v
```
or alternatively, you can just ignore the `enable` argument,
```
$ ansible-playbook playbooks/manage-dashboard-agents.yml  -e "environ=staging deploy_id=dashboard_mariadb_repl_c1" -v
```
If you need to uninstall it, simply specify `enable=false` which will disable the dashboard and its agents by uninstalling it in the cluster.
```
$ ansible-playbook playbooks/manage-dashboard-agents.yml  -e "environ=staging deploy_id=dashboard_mariadb_repl_c1 enable=false" -v
```

### Enable Query monitoring agents
Enabling the query monitor for [MySQL](https://docs.severalnines.com/docs/clustercontrol-staging/user-guide/gui-v1/mysql/query-monitor/) and for [PostgreSQL](https://docs.severalnines.com/docs/clustercontrol-staging/user-guide/gui-v1/postgresql/query-monitor/) behaves differently. Query monitor is only supported on a limited types of clusters (MySQL and PostgreSQL variant databases only) and implements different requirements for these type of database clusters. 

When you enable the query monitor, it simply deploys agents to all the database nodes that is supported by this feature. The deployment for query monitoring agents uses RPC V2 by simply invoking cURL command. In order to do this with **s9s-ansible**, we need to create the inventory file first and we name it as `querymonitor_mariadb_repl_c1`. Then save this file to `inventory/group_vars/staging/manage/` directory. Our params in this inventory file is just simple yet short and is shown below:
```
$ cat inventory/group_vars/staging/manage/querymonitor_mariadb_repl_c1
cluster_name: "mariadb-replication-c1"
cluster_type: replication
enable_qm: "{{ enable |d(true) }}"
rpc_user: myuser
rpc_password: myuserpass
rpc_v2_url: https://127.0.0.1:9501/v2/jobs
```
All we have is declare the `cluster_name`, `cluster_type` as the main important params or variables. Then we have `rpc_user` and `rpc_password` which this playbook operates using RPC V2 by invoking cURL command. Lastly, we have the `enable_qm` variable which means to enable (**true** as the default action if not stated) or simply pass the it with false as `enable_qm=false` when running the playbook. The playbook needed for this action is `manage-query-monitor.yml`. To enable the query monitoring agents, simply run
```
$ ansible-playbook playbooks/manage-query-monitor.yml  -e "environ=staging deploy_id=querymonitor_mariadb_repl_c1 enable=true" -v
```
or alternatively, you can just ignore the `enable` argument,
```
$ ansible-playbook playbooks/manage-query-monitor.yml  -e "environ=staging deploy_id=querymonitor_mariadb_repl_c1 enable=true" -v
```
If you need to uninstall it, simply specify `enable=false` which will disable the query monitoring agents by uninstalling it in the cluster.
```
$ ansible-playbook playbooks/manage-query-monitor.yml  -e "environ=staging deploy_id=querymonitor_mariadb_repl_c1 enable=false" -v
```

### Unregistering a node (HAProxy, ProxySQL, replica)
Unregistering HAProxy or ProxySQL or a database node (replica for this example) is the same approach where you apply this action using ClusterControl web UI. However, the great advantage when doing this with **s9s-ansible** is that, you can unregister multiple nodes by just specifying or listing it in the inventory file. Unlike if you do this action using ClusterControl UI, you have to deal with one node at a time and click the "Unregister Node" in the menu. That might be a hassle if you are doing multiple nodes to unregister which just helps you make the job done quicker.

In this section, we have the following nodes for example:
```
$ s9s nodes --list --long --cluster-id 92 --batch
soM- 10.5.17    92 mariadb-replication-c1 192.168.40.58 3306 Up and running (read-write).
yo-- 2.4.3      92 mariadb-replication-c1 192.168.40.58 6032 Process 'proxysql' is running.
ho-- 2.0.13     92 mariadb-replication-c1 192.168.40.58 9600 Process 'haproxy' is running.
Ao-- 1.0.0      92 mariadb-replication-c1 192.168.40.58 4433 Process 'cmnd' is running.
soS- 10.5.17    92 mariadb-replication-c1 192.168.40.59 3306 Up and running (read-only).
yo-- 2.4.3      92 mariadb-replication-c1 192.168.40.59 6032 Process 'proxysql' is running.
ho-- 2.0.13     92 mariadb-replication-c1 192.168.40.59 9600 Process 'haproxy' is running.
Ao-- 1.0.0      92 mariadb-replication-c1 192.168.40.59 4433 Process 'cmnd' is running.
coC- 1.9.4.5730 92 mariadb-replication-c1 192.168.40.6  9500 Up and running.
```
Our goal is that, we have to remove ProxySQL, HAProxy, and a replica node leaving only one node, the Primary which is the IP _192.168.40.58_ in this context.

In order to do this, let's create the inventory file and named it as `mariadb_repl_c1_ha_proxy_db`, which stands for the name of the cluster, then the type of nodes to be deleted which are HAProxy, ProxySQL and a DB node. Since the action is about unregister a node, we need to save this inventory file to `inventory/group_vars/staging/unregister` directory path. Take note that the file named `mariadb_repl_c1_ha_proxy_db` shall be our given `deploy_id` value when we run the playbook later. Below are the contents of this inventoy file,
```
$ cat inventory/group_vars/staging/unregister/mariadb_repl_c1_ha_proxy_db
cluster_name: "mariadb-replication-c1"
cluster_type: "replication"
nodes:
    - host: 192.168.40.58
      port: 9600
    - host: 192.168.40.59
      port: 9600
    - host: 192.168.40.58
      port: 6032
    - host: 192.168.40.59
      port: 6032
    - host: 192.168.40.59
      port: 3306
wait_log: "--wait --log"
uninstall: true
```
Let's explain a bit. First, we need to declare the `cluster_name`, its `cluster_type`, then the list of `nodes` in a dictionary format. We also specify the `uninstall` set to true which means that this will uninstall its system services used by the port. For example, port 9600 is for HAProxy, port 6032 is used by ProxySQL, and port 3306 is used by MySQL/MariaDB/Percona Server. This means that these packages shall be uninstalled. Then the `wait_log` is just an optional parameter which will print the logs and wait for every action pass to the cmon or ClusterControl using **s9s** command. Take note that `wait_log: "--wait --log"` is only applicable for **s9s** command, but not for RCP V2.

Now that we're ready to go, we need to invoke the playbook `unregister-node.yml`. To do this, let's run the following command:
```
$ ansible-playbook playbooks/unregister-node.yml -e "environ=staging deploy_id=mariadb_repl_c1_ha_proxy_db" -v
```

If you verify via ClusterContorl web UI or through the **s9s job** command, you can determine the nodes and verify it that these are being removed.
```
$ s9s job --list --cluster-id=92 --show-finished --batch|tail -5
10335 92 FINISHED test_dba admins 09:42:53            100% Remove '192.168.40.58' from the Cluster
10336 92 FINISHED test_dba admins 09:42:59            100% Remove '192.168.40.59' from the Cluster
10337 92 FINISHED test_dba admins 09:43:06            100% Remove '192.168.40.58' from the Cluster
10338 92 FINISHED test_dba admins 09:43:17            100% Remove '192.168.40.59' from the Cluster
10339 92 FINISHED test_dba admins 09:43:29            100% Remove '192.168.40.59' from the Cluster
```

Then let's verify how many nodes do we have remained in the cluster,
```
$ s9s nodes --list --long --cluster-id 92 --batch
soM- 10.5.17    92 mariadb-replication-c1 192.168.40.58 3306 Up and running (read-write).
Ao-- 1.0.0      92 mariadb-replication-c1 192.168.40.58 4433 Process 'cmnd' is running.
Ao-- 1.0.0      92 mariadb-replication-c1 192.168.40.59 4433 Process 'cmnd' is running.
coC- 1.9.4.5730 92 mariadb-replication-c1 192.168.40.6  9500 Up and running.
```
So we have the HAProxy, ProxySQL, and a replica being removed.
### Create a replica
Earlier or in the previous section, we have removed the replica. In this section, we'll try to add back again that replica. Doing this in **s9s-ansible** is simple. First, we create the inventory file and we'll call it `mariadb_repl_c1_replica59`, suffixing it with _replica59_ since I am going to add a replica with IP _192.168.40.59_. Then have to save this file to ` inventory/group_vars/staging/add_node` directory. The contents for this inventory file is simple,
```
cluster_name: "mariadb-replication-c1"
cluster_type: replication
vendor: "mariadb"
version: "10.5"
db_nodes:
    - host: 192.168.40.59
      port: 3306
      node_type: slave
wait_log: "--wait --log"
prefix: "mysql://"
```
All we have to do is specify the `cluster_name`, the `cluster_type`, the `vendor` and `version` of the database based on the exisitng cluster. Then the `db_nodes` which is in a dictionary format where the subitems consist of the `host`, `port`, `node_type` which is equivalent to its role (_slave_ for this example). Then specify the `prefix` which should have the value of `mysql://`. Lastly, the `wait_log` is just an optional parameter which will print the logs and wait for the action taken.

Now, let's add the replica by invoking the playbook `add-database-node.yml` as follows:
```
$ ansible-playbook playbooks/add-database-node.yml -e "environ=staging deploy_id=mariadb_repl_c1_replica59" -v
```

After a successfull run, verifying with **s9s nodes** command shows that the replica has been added.
```
$ s9s nodes --list --long --cluster-id 92 --batch
soM- 10.5.17    92 mariadb-replication-c1 192.168.40.58 3306 Up and running (read-write).
Ao-- 1.0.0      92 mariadb-replication-c1 192.168.40.58 4433 Process 'cmnd' is running.
Ao-- 1.0.0      92 mariadb-replication-c1 192.168.40.59 4433 Process 'cmnd' is running.
soS- 10.5.17    92 mariadb-replication-c1 192.168.40.59 3306 Up and running (read-only).
coC- 1.9.4.5730 92 mariadb-replication-c1 192.168.40.6  9500 Up and running.
```

### Drop the cluster
Dropping the cluster in **s9s-ansible** is simple but it has to be executed very carefully as this shall remove the existing cluster within ClusterControl. That means, the monitoring for database nodes, load balancers, and others (query monitor, Prometheus agents), shall be unregistered or removed in ClusterControl. However, that's all just the caveat since dropping the cluster is the same as dropping using ClusterControl UI but just invokes **s9s cluster** command to manage the dropping of the cluster. This means that, dropping does not drop or destroy any valuable data in the database, but only unregisters it within the ClusterControl.

To do this, let's create the inventory file named `mysql_repl_c1` and save this file to `inventory/group_vars/staging/drop_cluster` directory since we're dealing with dropping a cluster.

The inventory file `inventory/group_vars/staging/drop_cluster/mysql_repl_c1` contains only few params to do the job done. See below:
```
$ cat inventory/group_vars/staging/drop_cluster/mysql_repl_c1
cluster_name: "mariadb-replication-c1"
cluster_type: replication
wait_log: "--wait --log"
remove_backups: true
```
What you have to make sure is specify the `cluster_name`, `cluster_type`. The `wait_log` with value `--wait --log` is optional as well as the `remove_backups` is also optional. Not specifying the `remove_backups` parameter shall default to false. 

To drop the cluster, simply run the playbook `drop-cluster.yml` just like below:
```
$ ansible-playbook playbooks/drop-cluster.yml -e "environ=staging deploy_id=mysql_repl_c1" -v
```
Since it has been dropped, verifying through the list of Clusters doesn't show the `mariadb-replication-c1` clustername anymore. See below:
```
$ s9s cluster --list --long
ID STATE   TYPE              OWNER    GROUP  NAME           COMMENT
79 STARTED mongodb           test_dba admins mongodb-mdbgrp All nodes are operational.
82 STARTED replication       test_dba admins mysql-repl5.7  All nodes are operational.
83 STARTED postgresql_single test_dba admins postgres-c1    All nodes are operational.
Total: 3
```
