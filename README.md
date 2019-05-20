# vcs-fitcycle-deployer 
---------------------
---------------------
Version: 1.3

## Getting Started 

These instructions will allow you to get a copy of the project and deploy Fitcycle application in AWS. It will
also configure the instances. 

The app can be deployed with 2 different architectures

1. With MySql database on a VM and a HA Proxy to load balance between databases

2. AWS RDS as a database with or without multi-az mode (multi-az is recommended mode) 

See notes below for troubleshooting.

## Requirements:

Terraform Version: 0.12+ 

Ensure the correct AMIs are available in the region where the application needs to be deployed. Currently the AMIs
are available in the following region(s)

**Note: These AMIs are private. For access to these images, contact Bill Shetti or Shrivatsa Upadhye**

### Fitcycle AMIs

**us-east-1 (N. Virginia)**

    web="ami-0424ce05e6eac4d44"
    mgmt="ami-09b7afbfbed29099c"
    dblb="ami-0c287d8bb736b0dc4"
    db="ami-03442710b971503b5"
    app="ami-0c5a97dcec802ce81"
    api="ami-04aba6a14439a24d2"


**us-west-1 (N. California)**

    web="ami-029218bb923933d1a"
    mgmt="ami-048ef73aadc6c3bf6"
    dblb="ami-06eb4b1fd6d3fa7f7"
    db="ami-001af27175ceb5c1d"
    app="ami-0cbde20c9254f27cf"
    api="ami-013c6f058d5323ba3"


## High level Architecture

![Fitcycle Architecture](https://github.com/ishrivatsa/vcs-fitcycle-deployer/blob/master/fitcycle.png)


## Instructions

1. Clone this repository to your local system.

2. It contains 3 directories - `fitcycle_ansible` , `fitcycle_ansible_with_rds` and `fitcycle_terraform`. Change directory to ` fitcycle_terraform`

3. Create a file name `terraform.tfvars` and populate it with the values listed below. You can also add additional
variable values in this file (See Step 4)

Ensure that aws_vpc_cidr is a /8, /12, /16 network which is in accordance with RFC 1918

```

10.0.0.0 - 10.255.255.255 (10/8 prefix)

172.16.0.0 - 172.31.255.255 (172.16/12 prefix)

192.168.0.0 - 192.168.255.255 (192.168/16 prefix)

```

terraform.tfvars file

```
option_1_aws_access_key = " "    
option_2_aws_secret_key = " "
region = "us-east-1"

option_3_aws_vpc_name = "fitcycleDemo"
option_4_aws_vpc_cidr = "10.0.0.0/16"

product = "fitcycle"
team = "dev-team"
owner = "teamlead"
environment = "staging"
organization = "acmefitness"
costcenter = "acmefitness-eng"

```

4.[OPTIONAL] You may also set values for `option_5_aws_admin_ssh_key_name`, `option_6_aws_admin_public_ssh_key`, `option_7_aws_dev_ssh_key_name`, `option_8_aws_dev_public_ssh_key` within the `terraform.tfvars` file as 
shown below.

**Note that doing so can result in an error ` Existing Key Pair`, as AWS doesnot allow creation of ssh keys with
same key name.** 

**Alternative to this is to just provide the `option_6_aws_admin_public_ssh_key` and `option_8_aws_dev_punlic_ssh_key` in the `.tfvars` file and omit the `option_5_aws_admin_ssh_key_name` and `option_7_aws_dev_ssh_key_name`
. By doing so, everytime terraform is run, you can provide a new `ssh key name`**

```
option_1_aws_access_key = " "
option_2_aws_secret_key = " "
region = "us-east-1"

option_3_aws_vpc_name = "fitcycleDemo"
option_4_aws_vpc_cidr = "10.0.0.0/16"
option_5_aws_admin_ssh_key_name = "adminKey"
option_6_aws_admin_public_ssh_key = " PASTE YOUR PUBLIC SSH KEY HERE - file ending with id_rsa.pub"

option_7_aws_dev_ssh_key_name = "devKey"
option_8_aws_dev_public_ssh_key = " PASTE YOUR PUBLIC SSH KEY HERE - file ending with id_rsa.pub"


product = "fitcycle"
team = "dev-team"
owner = "teamlead"
environment = "staging"
organization = "acmefitness"
costcenter = "acmefitness-eng"
	
```
[OPTIONAL] If you need to use a different AMI ID(s), use the following `terraform.tfvars` file

In this example, we are updating the region and the AMI IDs for that specific region.

```
option_1_aws_access_key = " "
option_2_aws_secret_key = " "
region = "us-west-1"

images = {

    web="ami-029218bb923933d1a"
    mgmt="ami-048ef73aadc6c3bf6"
    dblb="ami-06eb4b1fd6d3fa7f7"
    db="ami-001af27175ceb5c1d"
    app="ami-0cbde20c9254f27cf"
    api="ami-013c6f058d5323ba3"
}

option_3_aws_vpc_name = "fitcycleDemo"
option_4_aws_vpc_cidr = "10.0.0.0/16"

product = "fitcycle"
team = "dev-team"
owner = "teamlead"
environment = "staging"
organization = "acmefitness"
costcenter = "acmefitness-eng"

```

5. If you plan on using ***remote backend***, such as S3, to store the state file, then Run 

`terraform init --backend-config="bucket=mybucket" --backend-config="key=path/to/my/key/some.tfstate" --backend-config="region=us-east-1"` 

**The S3 bucket should be pre-created to be used for the backend.**

**To use remote backend, Terraform will need List, Read and Put access to the bucket. Ensure that these permissions are added to the policy and assigned to the user that will be used.**

Fix any errors that are reported before proceeding.

If you plan on using *local backend* **[NOT RECOMMENDED]**, to store the state file, then edit the `provider.tf` file and comment the backend section.
 
Then run `terraform init`

6. Run `terraform plan -var-file=terraform.tfvars` to ensure there are no errors. Fix any errors before proceeding. 

7. Run `terraform apply -var-file=terraform.tfvars -state=terraform.tfstate` to deploy your infrastructure. 

**It's best to provide a state file path `-state=<FILE_PATH>` , if you are planing to deploy multiple instances 
of the entire infra**

**Alternatively, you may also run `terraform apply -var-file=terraform.tfvars -state=terraform.tfstate --auto-approve` . This will execute terraform without need for an additional approval step.**


**You can also execute deployment on different regions by using multiple `.tfvars` file and providing different inputs

For example : `terraform apply -var-file=terraform.tfvars -var-file=us_west_1_terraform.tfvars`


8. Enter the values for various variables when prompted

**For deployment with MySql on a VM and HA Proxy**

```
var.option_9_use_rds_database = 0
var.option_10_aws_rds_identifier = 0
var.option_11_multi_az_rds = 0

```

**For deployment with AWS RDS - single az**

```
var.option_9_use_rds_database = 1
var.option_10_aws_rds_identifier = rdsFitcycle
var.option_11_multi_az_rds = 0

```

**For deployment with AWS RDS - multi-az** [This deployment can take upto ~ 15 mins]

```
var.option_9_use_rds_database = 1
var.option_10_aws_rds_identifier = rdsFitcycle
var.option_11_multi_az_rds = 1

```

9. Once **Terraform** has successfuly completed execution, wait for coupe of minutes and then SSH into the management VM 
or the jumpbox. 

You can login into your AWS console to get the Public IP address for the Management (mgmt) box or you can run the
following command `terraform output`

The output should look like this

```
mgmt_public_ip = 52.90.92.175
vpc_id = vpc-04def571849hde0
web1_public_ip = 35.173.230.151
web2_public_ip = 35.173.211.14

```

10. The mgmt/jumpbox is pre-baked with the ansible templates. 

**Change the directory to `fitcycle_ansible`** for deployment with MySQL and HA Proxy 

**Change the directory to `fitcycle_ansible_with_rds`** for deployment with AWS RDS

11. Edit the file export_keys.sh and provide the details for AWS ACCESS KEY and AWS SECRET ACCESS KEY. Then 

Run the command `source export_keys.sh`

12. Update the `inventory/hosts.aws_ec2.yaml` file for the specific region in which the deployment occurs. 
 
```
plugin: aws_ec2
# Enter the region to deploy in
regions:
  - us-east-1

filters:
   instance-state-name: running
# Enter the VPC-ID from terraform outputs or aws console
   vpc-id:

```

13. Run this command

    **For MySql based deployment**

    `ansible-playbook -i inventory/hosts.aws_ec2.yaml configure_fitcycle.yml -e 'db_user=db_app_user db_password=VMware1!' -vvv`


    **For RDS based deployment**

     `ansible-playbook -i inventory/hosts.aws_ec2.yaml -i inventory/hosts.aws_rds.yaml configure_fitcycle.yml -e 'db_user=db_app_user db_password=VMware1!' -vvv`


14. Once ansible completes configuring successfully, you can go to a web browser and access the app with any of the 
public IP addresses of the **web** VM.


## Destroying the infrastructure

15.  Run the command `terraform destroy -var-file=terraform.tfvars -state=terraform.tfstate --auto-approve`
     - If prompted for any input variable provide the values.  This is currently a bug with terraform.


## Troubleshooting

### Unable to access http://{WEB_PUBLIC_IP} 

- Ensure that public IP addresses were assigned to the instances. 
- Ensure the Security Group allows port 80 from all source IP addresses (0.0.0.0/0)

### Unable to access http://{WEB_PUBLIC_IP}/api/v1.0/signups 

- Repeat step 12 and wait for a few minutes 
- Ensure that the url path is correct

### skipping: no hosts matched

- Repeat step 11 and Ensure the values for `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` are set.
- Run this command `ansible-inventory -i inventory/hosts.aws_ec2.yaml --list`
or 
- Run this command `ansible-inventory -i inventory/hosts.aws_ec2.yaml -i inventory/hosts.aws_rds.yaml --list`

- Repeat step 12

