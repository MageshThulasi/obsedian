# [Main Tools Documentation](https://docs.ipc.inday.io/onboarding/tools/)
[Sean Dugan's Tools Setup Guide](https://confluence.workday.com/display/AWSFED/Tools+Setup)
# okta2aws
* [Okta for AWS - Console Access](https://confluence.workday.com/display/CCOE/Okta+for+AWS+-+Console+Access)
* [Okta for AWS - CLI Access](https://confluence.workday.com/pages/viewpage.action?pageId=782646257)
* [okta2aws](https://ghe.megaleo.com/INFServices/okta2aws)

# AWS - To setup profiles
/Users/magesh.thulasi/code/github/ghe.megaleo.com/INFServices/terraform-base
To install dependencies: 
pip3 install -r requirements.txt

To populate ~/.aws/config file:
python3 generate-config.py boto

# Login to dev bastion
ssh -p 321 bastion-dev-test.ipc.inday.io

## To check user details
magesh.thulasi@bastion-dev-test:~$ id magesh.thulasi
uid=20530(magesh.thulasi) gid=100(users) groups=100(users),53811(infrastructure.servers),24916(infra-systems),56661(inf.ipc),8632(infrastructure.engineering),23559(splunk.web.inf.users),29657(infra-systems.pci)

## list_instances
A utility named `list_instances` to list all the instances in the env for a given role.

list_instances bastion

### blackbox
The blackbox terraform module - https://ghe.megaleo.com/Pi-PublicCloud-TF-Modules/blackbox/blob/master/vars.tf#L4 has a role blackbox

From the bastion instance, you can run `list_instances blackbox`
magesh.thulasi@bastion-dev-test:~$ list_instances blackbox
```
+-----------------+
|       ip        |
+-----------------+
|  10.205.151.40  |
|  10.205.23.94   |
+-----------------+
```

You can then `sesh <ip_address>` and choose `inf.ipc` role.

aws-tunnel usw2-ipc-0
vault login -method=ldap

---
# General
Date: 11/13/23
Team member: Drishya Shyna

#pi_site_ops and #ask-bt 
aws s3 ls --profile alm

sso-cli aws
sso-cli aws console

# Accounts
There are 3 kinds of accounts, viz., hubs, spokes and others. *Spokes and (others) consume services from the Hubs.* Check out **[High Level IPC Service Offering](https://urldefense.com/v3/__https://workday.zoom.us/rec/share/60FEmSJ-pMiezW-v3SiUQD8QVzRyg4kynYVj7s3Jz649FqlXi21ubhJye-yQnjMr.r7ccHrCYlDzZOe4P__;!!Iz9xO38YGHZK!9PYfaYsgMXUm7ZhM95DpaMi5V3oniFSrzkReUXyGRo3Nc0Ua70OJa2sg3VV7cC1ReWgyaUX50VzBqPLinp0Dvw$)**

1. Hubs - is where all the INF services like vault are running. Example - ipc-1
2. Spokes - these are scylla envs that host SPC or SWC clusters. Example - wd105
3. Others - are splunk, etc accounts. Example - platsec-1, netsec-1, etc

The folders can be found at  /Users/magesh.thulasi/code/github/ghe.megaleo.com/INFServices/terraform-base/environments
# To connect to any given instance

## #1: Tunnel into the environment
```sh
aws-tunnel usw2-ipc-0
```
where usw2-ipc-0 is the environment

You can get the list of environments using
```sh
aws-tunnel -l
```

## #2: List the instances for a role
Change directory to the environment and execute list_instances
```sh
cd /Users/magesh.thulasi/code/github/ghe.megaleo.com/INFServices/terraform-base/environments/usw2-ipc-0
list_instances oidc
list_instances vault
```

This will show you the instances associated with this role:
```
------------------------------------------------------------------------------
|                              DescribeInstances                             |
+------------+-----------------------+---------------+--------+--------------+
|     az     |      instance_id      |      ip       | role   |    stack     |
+------------+-----------------------+---------------+--------+--------------+
|  us-west-2a|  i-0d31ae5167ade6f75  |  10.209.3.154 |  vault |  usw2-ipc-0  |
|  us-west-2b|  i-0dfdd97df92ee70ce  |  10.209.3.178 |  vault |  usw2-ipc-0  |
+------------+-----------------------+---------------+--------+--------------+
```
**Alternatively, you can get the IP address from the AWS console.**

### Check info about the environment

View the contents of the *stack.auto.tfvars.json* file for info such as account_id, cidr_block, region, etc. Below is a sample for env *usw2-ipc-0*

```json
{
  "account_id": "984582732533",
  "availability_zones": [
    "us-west-2a",
    "us-west-2b",
    "us-west-2c"
  ],
  "cidr_block": "10.209.2.0/23",
  "platsec_trust": "awsdev",
  "profile": "devinfservices1-admin",
  "region": "us-west-2",
  "class": "dev",
  "stack": "usw2-ipc-0",
  "hub": "usw2-ipc-0",
  "stack_commonid": "416828578396",
  "source_dc": "pdx-eng"
}
```
## #3: Login to the instance using sesh

```sh
sesh 10.209.3.154
```

## #4: Login to AWS UI Console
1. Open the AWS UI Console using `sso-cli aws console`
2. Copy the account_id from the *stack.auto.tfvars.json* above
	1. Open the ~/.aws/config file and locate the account_id
	2. Copy the role_arn
	3. Use this account_id and role to switch role in the AWS UI Console

# Vault
1. Connect to GlobalProtect - US Secure East
2. aws-tunnel -L usw2-ipc-0
3. curl https://vault.services.wd/v1/sys/health
4. vault status -address=https://vault.services.wd
