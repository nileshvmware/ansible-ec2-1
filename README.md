# quick-ec2-instance

This project is intended to act as a simple example of spinning up an AWS instance, in preparation for additional automation, for example to deploy Satellite, Openshift, Pacemaker or similar.

The instructions and playbooks in this repository:
- Configure some AWS credentials in the Ansible Vault
- Prepare the shell environment for the AWS dynamic inventory script
- Create a default VPC if one does not already exist (examples for specific VPC creation are included too where explicitly defined subnets, security groups, internet gateways, routing tables or similar are required.)
- Create an appropriately sized instance (the file system layout is kept deliberately simple, for production deployments refer to the corresponding documentation)

## Contents

```
.
├── ansible.cfg
├── inventories
│   ├── aws
│   ├── ec2.ini
│   └── group_vars
│       └── all
│           ├── vars
│           │   └── all.yml
│           └── vault
│               └── all.yml
├── playbooks
│   ├── create-arm-ec2-instances.yml
│   ├── create-default-infrastructure.yml
│   ├── create-ec2-instances.yml
│   ├── create-specific-infrastructure.yml
│   ├── create-x86-ec2-instances.yml
│   ├── debug-infrastructure.yml
│   ├── destroy-ec2-instances.yml
│   └── templates
│       ├── config.j2
│       └── credentials.j2
└── README.md
```

- `inventories/aws`: AWS dynamic inventory script
- `inventories/group_vars/all/vars/all.yml`: inventory group vars for AWS and RHSM
- `inventories/group_vars/all/vault/all.yml`: sensitive inventory group vars for AWS and RHSM
- `playbooks/create-default-infrastructure.yml`: idempotently create a default AWS VPC 
- `playbooks/create-specific-infrastructure.yml`: idempotently create specific VPN with explicit subnets, security groups, internet gateways and routing tables
- `playbooks/create-x86-ec2-instances.yml`: create x86_64 EC2 instance(s)
- `playbooks/create-arm-ec2-instances.yml`: create arm EC2 instance(s)
- `playbooks/destroy-ec2-instances.yml`: destroy the EC2 instance(s)
- `playbooks/debug-infrastructure.yml`: debug output from VPC facts

## Variables

- `ec2_x86_image`: AMI image ID to provision EC2 instances from (default ami-0e12cbde3e77cbb98) 
- `ec2_region`: AWS region to create VPC/provision instances (default: eu-west-1)
- `vaulted_aws_access_key`: AWS Access key ID (https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys)
- `vaulted_aws_secret_key`: AWS Secret Access Key (as above)
- `vaulted_ec2_keypair`: AWS EC2 keypair (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

## Steps
1. Clone the repository
```
$ git clone git@github.com:wmcdonald404/quick-ec2-instance.git ~/quick-ec2-instance/
```
2. Define your AWS access_key, secret_key and keypair variables in a vault file

Note: vault/all.yml will contain the vaulted/encrypted values.  vars/all.yml is effectively a layer of redirection so that there is a plain-text copy of the variable name to aid troubleshooting/tracing.

```
$ mkdir -p ~/quick-ec2-instance/inventories/group_vars/all/{vars,vault}

$ ansible-vault create ~/quick-ec2-instance/inventories/group_vars/all/vault/all.yml
$ ansible-vault edit ~/quick-ec2-instance/inventories/group_vars/all/vault/all.yml

vaulted_aws_access_key: <access_key>
vaulted_aws_secret_key: <secret_key>
vaulted_ec2_key_pair: <key_pair_name>
```
3. Optionally, create a vault_pass file (nb: never check this into SCM)
```
vi ~/.vault_pass
```
4. Export environment variables required for the AWS dynamic inventory script
```
export AWS_ACCESS_KEY_ID=<access_key_id>
export AWS_SECRET_ACCESS_KEY=<secret_access_key>
```
5. Set appropriate variables for instance_count and instance_tag:
```
$ vi ~/quick-ec2-instance/inventories/group_vars/all/vars/all.yml 

ec2_instance_count: 1
ec2_instance_tag:
  name: satellite
```

6. Create AWS default VPC infrastructure
```
$ cd ~/quick-ec2-instance
$ ansible-playbook ~/quick-ec2-instance/playbooks/create-default-infrastructure.yml
```
7. Refresh the inventory cache (just in case this is an iteration run and a previous instance has been cached)
```
$ ~/quick-ec2-instance/inventories/aws --refresh-cache
```
8. Create EC2 Satellite instance
```
$ cd ~/quick-ec2-instance
$ ansible-playbook ~/quick-ec2-instance/playbooks/create-x86-ec2-instances.yml
```
9. Note the item.dns_name value

## Further Steps
Once a VPC and instance(s) are provisioned further automation, for example https://github.com/sean797/iac-satellite, may be used to deploy Satellite. The playbooks in that repository will:

- Subscribe the host appropriately
- Install Satellite
- Optionally: create some standard Satellite objects (products, content views, activation keys etc.)

