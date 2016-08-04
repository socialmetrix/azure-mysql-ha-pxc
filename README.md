<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fazure-quickstart-templates%2Fmaster%2Fmysql-ha-pxc%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fmysql-ha-pxc%2Fazuredeploy.json" target="_blank">
  <img src="http://armviz.io/visualizebutton.png"/>
</a>

This template lets you create a 3 node Percona XtraDB Cluster 5.6 on Azure.  It's tested on Ubuntu 12.04 LTS and CentOS 6.5.  

To verify the cluster, type in "mysql -h your_public_ip_dns_name -u test -p".  The password is what you set for sstuser in my.cnf. Run MySQL command "show status like 'wsrep%'".  You should see that wsrep_cluster_size is 3 by default and wsrep_ready is "ON". 

To gain root access to MySQL, you must ssh into a node and sudo to run mysql.   


##Preparation
1. Select a Ubuntu Version:
```bash
azure vm image list eastus2 canonical ubuntuserver
```

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
  --resource-group mysql-pxc smxpxc
``

## Launch Template Using Command Line
```bash
cp -pR my.cnf.template ~/Dropbox/Public/

azure group deployment create \
  --template-file azuredeploy.json \
  --parameters-file azuredeploy.parameters.json \
  mysql-pxc mysql-pxc
  
# if something went wrong:
# azure group delete -q mysql-pxc
```