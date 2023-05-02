# aws\_vpc

This role creates an AWS VPC with an internet gateway, one or more subnets, and one or more security groups.  By default, the VPC will contain a single public subnet and one security group for allowing public SSH connections. The variables are available to override the subnet and security group configuration.  The VPC and its network components can be easily deleted with a teardown option.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| aws\_vpc\_set\_role\_output | false | See the [Role Output](#role-output) section below. |
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

## Role Output

The `aws\_vpc\_set\_role\_output` variable can be set to "true" in order to capture the VPC resources created by the role.  The data is captured in a variable called `_aws_vpc_role_output`, which can then be used in tasks or plays in a playbook where the aws\_vpc role is used.  The `role_output` variable is a dictionary that will contain the following keys:

- vpc: data on the VPC created by the role, from the [amazon.aws.ec2\_vpc\_net\_info](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_vpc_net_info_module.html#return-vpcs/cidr_block) module
- igw: data on the internet gateway created by the role, from the [amazon.aws.ec2\_vpc\_igw\_info](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_vpc_igw_info_module.html#return-internet_gateways/attachments) module
- subnets: data on the subnets created by the role, from the [amazon.aws.ec2\_vpc\_subnet\_info](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_vpc_igw_info_module.html#return-internet_gateways/attachments) module
- security\_groups: data on the security groups created by the role, from the [amazon.aws.ec2\_security\_group\_info](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_vpc_igw_info_module.html#return-internet_gateways/attachments) module

## Using the role

Example role:

```
---
- name: Create and use an AWS VPC
  hosts: localhost

  tasks:
    - name: Create VPC named "{{ aws_vpc_name }}"
      ansible.builtin.include_role:
        name: jce_redhat.cloud_infra.aws_vpc
      vars:
        aws_vpc_name: ansible-test-vpc-1
        aws_vpc_owner: test-user
        aws_vpc_cidr: 172.22.0.0/16
        aws_vpc_subnets:
          - cidr: 172.22.0.0/24
            public: true
            tags:
              subnet_type: public
          - cidr: 172.22.50.0/24
            public: false
            tags:
              subnet_type: private
        aws_vpc_set_role_output: true

    - name: Get resource IDs from role output
      ansible.builtin.set_fact:
        my_vpc_id: '{{ _aws_vpc_role_output.vpc.vpc_id }}'
        my_igw_id: '{{ _aws_vpc_role_output.igw.internet_gateway_id }}'
        my_subnet_ids: '{{ _aws_vpc_role_output.subnets | map(attribute="subnet_id") }}'
        my_secgroup_ids: '{{ _aws_vpc_role_output.security_groups | map(attribute="group_id") }}'

    # do things with the IDs here

...
```

See the [collection playbooks](../../playbooks/) for other examples.
