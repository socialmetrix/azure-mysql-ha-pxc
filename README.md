# Azure Percona XtraDB Cluster Template
This template lets you create a 3 node Percona XtraDB Cluster 5.6 on Azure. 
It is based on [azure-quickstart-templates/mysql-ha-pxc](https://github.com/Azure/azure-quickstart-templates/tree/master/mysql-ha-pxc).

There are few changes to fit our use case, like:

- the Subnet is in a different Resource Group
- Upgrade the Ubuntu Version to **14.04.4-LTS**. I tried to use 16.04.0-LTS but run on issues when machine reboots.
- Enable boot diagnostics


<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsocialmetrix%2Fazure-mysql-ha-pxc%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fsocialmetrix%2Fazure-mysql-ha-pxc%2Fmaster%2Fazuredeploy.json" target="_blank">
  <img src="http://armviz.io/visualizebutton.png"/>
</a>

##Dependencies
This template depends on:

- **Resource Group:** `mysql-ha-pxc`
- **Subnet** of VNET `smxnetworks` called: `mysql-pxc` that holds the address range: `10.3.100.0/24`
- An **Storage Account** `smxmysqlhapxc`


1. Create a Resource Group: 
```bash
azure group create --tags 'billing=echo' mysql-ha-pxc eastus2
```

1. Create a Storage Account (Premium Storage): 
```bash
azure storage account create \
  --location eastus2 \
  --sku-name PLRS \
  --kind Storage \
  --resource-group mysql-ha-pxc smxmysqlhapxc
```

## Launch Template Using Command Line
```bash
azure group deployment create \
  --template-file azuredeploy.json \
  --parameters-file azuredeploy.parameters.json \
  mysql-ha-pxc mysql-ha-pxc
  
```

##Using the cluster

To verify the cluster, type in `mysql -h your_public_ip_dns_name -u test -p`.  
The password is defined for sstuser in `/etc/my.cnf`. 
Run MySQL command `show status like 'wsrep_cluster_size';`.  You should see that wsrep_cluster_size is 3 by default and wsrep_ready is "ON".

```
mysql> show status like 'wsrep_cluster_size';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```

To gain root access to MySQL, you must ssh into a node and sudo to run mysql.   

##Debug
Script will be downloaded on each machine at: `/var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.4.1.0/download/0`

It runs with those params:
`bash azurepxc.sh 10.3.100.11,10.3.100.12,10.3.100.13 10.3.100.11 bootstrap-pxc https://raw.githubusercontent.com/socialmetrix/azure-mysql-ha-pxc/master/my.cnf.template`

```
source azurepxc.sh

export CLUSTERADDRESS=10.3.100.11,10.3.100.12,10.3.100.13
export NODEADDRESS=10.3.100.13
export NODENAME=$(hostname)
export MYSQLSTARTUP=bootstrap-pxc
export MYCNFTEMPLATE=https://raw.githubusercontent.com/socialmetrix/azure-mysql-ha-pxc/master/my.cnf.template
export SECONDNIC=

```

Now I can execute each individual functions, having the env variables correctly configured.

