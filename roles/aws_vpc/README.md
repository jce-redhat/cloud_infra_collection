# aws\_vpc

This role creates an AWS VPC with an internet gateway, one or more subnets, and one or more security groups.  By default, the VPC will contain a single public subnet and one security group for allowing public SSH connections. The variables are available to override the subnet and security group configuration.  The VPC and its network components can be easily deleted with a teardown option.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| aws\_vpc\_name | ansible-built-vpc | Name of the VPC.  The name is also used in a tag called `vpc_name` in the rest of the VPC network components. |
| aws\_vpc\_region | us-west-2 | The AWS region where the VPC is created |
| aws\_vpc\_owner | \_UNSET\_ | Used in the `owner` tag for the VPC and its network resources.  If running the role from the automation controller, the `awx_user_name` variable will override this value. |
| aws\_vpc\_cidr | 10.249.0.0/16 | The CIDR to use for the VPC |
| aws\_vpc\_tags | {} | Additional tags to apply to the VPC |
| aws\_vpc\_teardown | false | If set to "true", will delete the named VPC and its network resources. The resources with the `vpc_name` tag that matches the supplied `aws_vpc_name` variable will be deleted. |
| aws\_vpc\_igw\_tags | {} | Additional tags to apply to the internet gateway |
| aws\_vpc\_subnets | <pre>- cidr: 10.249.0.0/24<br>  public: true</pre> | A list of dictionaries of subnets to create.  If the `public` key is present and set to a boolean true value, any instances created in the subnet will have a public IP address assigned. An availability zone can be specified with the `az` key. |
| aws\_vpc\_security\_groups | <pre>- name: public-ssh<br>  rules:<br>    - proto: tcp<br>      from\_port: 22<br>      to\_port: 22<br>      cidr\_ip: 0.0.0.0/0</pre> | A list of dictionaries of security groups to create. The `rules` key must be a list of rules that can be passed to the [amazon.aws.ec2\_security\_group](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_security_group_module.html#parameter-rules) Ansible module. |

## Resource tags

By default, the VPC and its internet gateway, subnet(s), and security group(s) will have the following tags applies:

| Tag | Value |
|-----|-------|
| vpc\_name | The value of the `aws_vpc_name` role variable |
| owner | The value of either the `awx_user_name` variable from the automation controller (if set), or the `aws_vpc_owner` role variable |

Additional tags for each resource can be applied with the relevant variables, see below for details.

**NOTE:** the `vpc\_name` tag is used for Ansible task idempotency.  If another VPC is created with the same name and same tag, it could have unintended side effects, especially when using the VPC teardown feature.

## Using the role

See the [build-vpc.yml](../../playbooks/build-vpc.yml) example playbook.
