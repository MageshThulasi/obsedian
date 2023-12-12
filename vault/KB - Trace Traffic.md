# Hub & Spoke Traffic Patterns #kt/raymond/traffic
- [ ] [Traffic Flow Patterns - Spoke to Hub](https://urldefense.com/v3/__https://workday.zoom.us/rec/share/B69674Ask79U0fdaRjhBbfTFJzrA3YtJK1mRb5AHF4DVUB2mWbX0PluGnTxCBdkM.pUyMF7kM90ktzAkG__;!!Iz9xO38YGHZK!-UvGIVMGDe78QMz9i2YqawFWns9VTEji9DFXLo4kUsI1A2lgEeG34FDTXZH9fDwRT8tEZpVkZceyWADJVrb8rA$)
	- [ ] Passcode: y78R1V!l
- [ ] [Private Link/VPC endpoint interface](https://docs.aws.amazon.com/whitepapers/latest/aws-privatelink/creating-highly-available-endpoint-services.html) #kb/vault/vpc-private-link
# Traffic Flow - Spoke to Hub #kb/vault/traffic

## Overview
![[Traffic Flow - Spoke to Hub.png|white]]

- [ ] Spoke: euc1-wd103-prod
- [ ] Hub: euc1-ipc-1
- [ ] Tools: aws-tunnel, sso-cli, list_instances and sesh
## Spoke side: Trace vault service #kb/vault/traffic/spoke

* Login to bastion-ssm instance of the spoke
```shell
aws-sessioin euc1-wd103-prod
dig +short vault.services.wd
```
* Pick one of the IP addresses from the dig command and trace it to VPC Endpoint Network Service.
	* Get the account and region details from ==stack.auto.tfvars.json== in the spoke's environments folder

> sso-cli aws console

In the AWS UI:
1. Navigate to Network Interfaces and filter by ==10.210.250.6==
2. From the Description field, copy the VPC endpoint interface name
3. Navigate to VPC | Endpoints and filter 'VPC Endpoint ID = vpce-0113c8e4a4692156b'
4. Copy the Service name field. It would look like =='com.amazonaws.vpce.eu-central-1.vpce-svc-07c4014cee13f13fa'==
## Hub side: Trace vault service #kb/vault/traffic/hub
In the AWS UI, switch to the hub account
* Navigate to VPC } Endpoint services and filter 'Service name = com.amazonaws.vpce.eu-central-1.vpce-svc-07c4014cee13f13fa'
	* Notice the name ending with =='haproxy-ingress'==
* Go to Load balancers tab and click on one of the 3 Load balancer names
* In the Load balancers window, go to Listeners tab and click on the Forward to target group link
* You will see 3 instances for haproxy-ingress. Click on one of the instances and you would have arrived at the EC2 instance hosting the HAProxy. Take a note of the Private IPv4 address
### Tracing from the HAProxy instance #kb/vault/traffic/haproxy
Login to bastion-ssm instance of the spoke
```shell
aws-session euc1-ipc-1
list_instances haproxy-ingress
```
> The list_instances will output 3 IP addresses and one of them will match with the Private IPv4 address from the previous section

Using vault role: infra-systems
```shell
sesh 10.209.4.87
pa aux | grep haproxy
vi /etc/haproxy/haproxy.cfg
```
The config will show you the **backend b_vault** resolves via consul, which is defined in the section ==resolvers consul==, which runs on port 8600
```shell
# To check what is running on port 8600 (consul)
sudo netstat -tulpn | grep 8600
# To find out the IP address for vault service discovered by consul
dig @127.0.0.1 -p 8600 _vault._v1.service.consul ALL
```
These IP addresses point to the EC2 instance where the vault is running
### Inspecting the vault instance #kb/vault/traffic/vault-instance
You can now sesh into one of the IP addresses from the consul discovery in the previous section. Remember to choose the **'infra-systems'** role

> sesh 10.209.5.157

This is the instance where the vault is running. #kb/linux/commands
```shell
ps aux | grep vault

cat /etc/os-release
hostnamectl
uname -r

cat /etc/vault-agent.hcl
cat /etc/vault/config.json | jq

# To check what is running on port 8600 (consul)
sudo netstat -tulpn | grep 8600
# To find out the IP address for vault service discovered by consul
dig @127.0.0.1 -p 8600 _vault._v1.service.consul ALL
```
```shell
magesh.thulasi@vault-usw2-ipc-0:~$ cat /etc/os-release
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
VERSION_ID="2"
PRETTY_NAME="Amazon Linux 2"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2"
HOME_URL="https://amazonlinux.com/"
SUPPORT_END="2025-06-30"
```
AMI Versions:
- AMI-1 - PRETTY_NAME="Amazon Linux AMI 2018.03"
- AMI-2 - PRETTY_NAME="Amazon Linux 2"
- AMI-3 - PRETTY_NAME="Amazon Linux 2023"

---
TODO: What is the relationship here? #todo
```shell
magesh.thulasi@bastion-ssm-euc1-ipc-1:~$ dig +short vault.services.wd
10.209.5.196
10.209.5.184
10.209.5.132
magesh.thulasi@bastion-ssm-euc1-ipc-1:~$ list_instances vault
+-----------------+
|       ip        |
+-----------------+
|  10.209.5.157   |
|  10.209.5.221   |
+-----------------+
magesh.thulasi@bastion-ssm-euc1-ipc-1:~$ list_instances haproxy-ingress
+-----------------+
|       ip        |
+-----------------+
|  10.209.4.168   |
|  10.209.5.123   |
|  10.209.4.87    |
+-----------------+
```
---
# Tracing usw2-ipc-0 #kb/vault/traffic/legacy
```shell
aws-tunnel usw2-ipc-0Sp

dig +short vault.services.wd
10.209.3.138
10.209.3.173
10.209.3.221
```
```
list_instances vault

+------------+-----------------------+---------------+--------+--------------+
|     az     |      instance_id      |      ip       | role   |    stack     |
+------------+-----------------------+---------------+--------+--------------+
|  us-west-2a|  i-0ed16b57c2d806dd2  |  10.209.3.150 |  vault |  usw2-ipc-0  |
|  us-west-2b|  i-0525e99e0c128c2ee  |  10.209.3.166 |  vault |  usw2-ipc-0  |
+------------+-----------------------+---------------+--------+--------------+
```
## Inspect the VPC interface
There is A record for the IP addresses from the above dig command. Login to the AWS UI Console using `sso-cli aws console`

1. Copy the account_id from the *stack.auto.tfvars.json*
	1. Open the ~/.aws/config file and locate the account_id
	2. Copy the role_arn
	3. Use this account_id and role to switch role in the AWS UI Console
2. Navigate to *Network Interfaces* and filter by the IP address. You can notice that it is a VPC Network Interface
3. Let us dive into the details of the VPC Network Interface
	1. Copy the Endpoint Id only from the Description field. It would look like *vpce-0711da0b46afa5d95*. The entire field would look like *VPC Endpoint Interface vpce-0711da0b46afa5d95*
	2. Open a new tab and navigate to VPC | Endpoints
	3. Filter by the Endpoint Id from step 1
	4. You can validate that the name of the Endpoint has vault in it. Example: *inf-usw2-ipc-0-vault*
	5. The next hop is to the VPC service. Copy the service name - *com.amazonaws.vpce.us-west-2.vpce-svc-06b164e031bb28897*
## Inspect the VPC service

Open the aws console in the new tab

1. Navigate to VPC | Endpoint services and filter by Service name = com.amazonaws.vpce.us-west-2.vpce-svc-06b164e031bb28897
2. Validate the details pane - for Details tab and Load balancers tab.
3. In the Load balancers tab, you will notice 3 LBs - one for each AZ
4. Click on one of the LBs and go to the Listeners tab
5. You will notice Protocol:Port = *TCP:443* and Default action = 'Forward to target group vault20200..'
6. Click on the link for Forward to target group
7. In the Targets tab, you will see two instances
8. Click one of them to take you to the Instances pane
9. In the Details tab, notice the Private IP = 10.209.3.166. This will one of the IPs from the list_instances earlier
## Inspect the Hub
This command will log you into a bastion of usw2-ipc-0 hub
> aws-session usw2-ipc-0

You can use the list_instances command here
> list_instances vault

You can now sesh into one of the IP addresses. Remember to choose the **'infra-systems'** role

> sesh 10.209.3.166

This is the instance where the vault is running. #kb/linux/commands
```shell
ps aux | grep vault

cat /etc/os-release
hostnamectl
uname -r

cat /etc/vault-agent.hcl
cat /etc/vault/config.json | jq

# To check what is running on port 8600 (consul)
sudo netstat -tulpn | grep 8600
# To find out the IP address for vault service discovered by consul
dig @127.0.0.1 -p 8600 _vault._v1.service.consul ALL
```
```shell
magesh.thulasi@vault-usw2-ipc-0:~$ cat /etc/os-release
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
VERSION_ID="2"
PRETTY_NAME="Amazon Linux 2"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2"
HOME_URL="https://amazonlinux.com/"
SUPPORT_END="2025-06-30"
```
AMI Versions:
- AMI-1 - PRETTY_NAME="Amazon Linux AMI 2018.03"
- AMI-2 - PRETTY_NAME="Amazon Linux 2"
- AMI-3 - PRETTY_NAME="Amazon Linux 2023"

---

# Trace traffic from a spoke (euc1-wd103-prod) to hub (euc1-ipc-1)

## Spoke side
1. Use dig to find the vault IP addresses
2. Network Interfaces - filter by one of the IP addresses
3. Fetch the EndpointId from the description field
4. Navigate to VPC | Endpoints and filter by the EndpointId
5. Copy the service name - *com.amazonaws.vpce.eu-central-1.vpce-svc-07c4014cee13f13fa*

Spoke service -> Network Interface -> VPC Endpoint Interface

## Hub side
1. Switch to the Hub account (euc1-ipc-1)
2. Navigate to VPC | Endpoint services and filter by the service name from step 5 of the previous section. You will notice that this is labeled 'haproxy-ingress'
3. 

Private Link -> VPC Endpoint Service -> Network Load Balancer -> HAProxy (and Consul) -> Vault (in a auto-scaling group)
```shell
aws-session euc1-wd103-prod

magesh.thulasi@bastion-ssm-euc1-wd103-prod:~$ dig +short vault.services.wd
10.210.245.151
10.210.250.6
10.210.240.144
```
```shell
aws-session euc1-ipc-1

magesh.thulasi@bastion-ssm-euc1-ipc-1:~$ dig +short vault.services.wd
10.209.5.184
10.209.5.132
10.209.5.196

FUNCTION: euc1-ipc-1 - eu-central-1b - bastion-ssm
magesh.thulasi@bastion-ssm-euc1-ipc-1:~$ list_instances vault
+-----------------+
|       ip        |
+-----------------+
|  10.209.5.157   |
|  10.209.5.221   |
+-----------------+
```

