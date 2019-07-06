## Update hosts file

sudo vim /etc/hosts

192.168.10.10   node1.example.com
192.168.10.11   node2.example.com

## Install Corosync and Pacemaker

sudo apt install corosync pacemaker pcs

## Enable start on boot

sudo systemctl enable pcsd

## Change password `hacluster` users (the same in all nodes)

sudo passwd hacluster

## Auth nodes (run in any node)

sudo pcs cluster auth node1.example.com node2.example.com -u hacluster -p password_here --force

## Create new cluster

sudo pcs cluster setup --name examplecluster node1.example.com node2.example.com 

## Cluster start on boot

```
sudo pcs cluster enable --all
sudo pcs cluster start --all
```

## Configuring Cluster Options

```
sudo pcs property set stonith-enabled=false
sudo pcs property set no-quorum-policy=ignore
sudo pcs property list
```
