# aws\_rhel\_instance

This role creates a RHEL instance in AWS using the amazon.aws.ec2\_instance module.  The role is meant to be used for demo purposes, as it creates a simple instance with a root disk and single network connection and does not provide the ability to add more volumes, change CPU settings, or other advanced features.

## Role Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| aws\_rhel\_instance\_name | Yes | rhel-test | Name of the RHEL instance |
| aws\_rhel\_instance\_type | Yes | t2.micro | The instance type to use for the instance |
| aws\_rhel\_region | Yes | us-west-2 | The AWS region where the instance is created |
| aws\_rhel\_subnet\_id | Yes | "" | The VPC subnet ID where the instance should be created.  The default is blank so this variable must be defined when calling the role. |
| aws\_rhel\_key\_name | No | "" | The name of the SSH key pair used to access the instance.  This variable is not required, but if it is not provided, SSH access to the instance will not be available. |
| aws\_rhel\_security\_groups | No | `default` | A list of security group names or IDs to attach to the instance.  If none are provided, the default security group in the VPC will be applied. |
| aws\_rhel\_tags | No | {} | A dictionary of tags to apply to the instance |
| aws\_rhel\_os\_version | Yes | rhel-test | The version of RHEL to use for the instance.  Choices are "rhel-7", "rhel-8", or "rhel-9", and the name is used to locate the AMI to use for the instance. |
| aws\_rhel\_ami\_arch | Yes | x86\_64 | The CPU architecture for the RHEL AMI |
| aws\_rhel\_ami\_filter | Yes | <pre>aws\_rhel\_ami\_filter:<br>  rhel-7: 'RHEL-7.9\_\*Hourly2\*'<br>  rhel-8: 'RHEL-8.7.0\_\*Hourly2\*'<br>  rhel-9: 'RHEL-9.1.0\_\*Hourly2\*'</pre> | The filter string used to find the RHEL AMI. See [below](#filtering-for-a-rhel-ami) for additional information. |
| aws\_rhel\_image\_id | No | undef | The specific AWS image ID to use when creating the instance.  This variable overrides the `aws_rhel_os_version` variable and the filtering mechanism used to find the latest RHEL AMI. |

# Filtering for a RHEL AMI

The role uses the amazon.aws.ec2\_ami\_info module to find RHEL AMIs based on a set of filters defined in the `aws_rhel_ami_filter` variable.  The default filters will search for the latest RHEL MarketPlace AMIs by name, although this variable can be overridden to search for any AMIs based on a string search of  the AMI name.  The keys of the `aws_rhel_ami_filter` variable must match the value provided in the `aws_rhel_os_version` variable.  If the `aws_rhel_image_id` variable is set, it overrides anything found with the filter mechanism.

For example, if you are using the Red Hat [Cloud Access Program(https://www.redhat.com/en/technologies/cloud-computing/cloud-access), the `aws\_rhel\_ami\_filter` might look like the following:

```
aws_rhel_ami_filter:
  rhel-7: 'RHEL-7.9_*Access2*'
  rhel-8: 'RHEL-8.7.0_*Access2*'
  rhel-9: 'RHEL-9.1.0_*Access2*'
```

For using a specific RHEL 8 AMI, the variable might be:

```
aws_rhel_ami_filter:
  rhel-8: 'RHEL-8.4.0_HVM-*-Hourly2-GP2'
```

Note that the CPU architecture should not be defined in the filter, and instead should be provided with the `aws_rhel_ami_arch` variable.

See the following for more information on the RHEL AMIs that are available:

[Red Hat Enterprise Linux Images (AMI) Available on Amazon Web Services (AWS)](https://access.redhat.com/solutions/15356) (NOTE: this page may lag behind the actual AMIs available, so filtering for the latest is preferred).
[What are the different types of RHEL AMIs available in AWS?](https://access.redhat.com/articles/3692431)
[How to Locate Red Hat Cloud Access Gold Images on AWS EC2](https://access.redhat.com/articles/2962171)

## Using the role

Example role:

```
---
- name: Create AWS RHEL instances
  hosts: localhost

  tasks:
    - name: Create RHEL instance in a specific VPC subnet
      ansible.builtin.include_role:
        name: jce_redhat.cloud_infra.aws_rhel_instance
      vars:
        aws_rhel_instance_name: test-instance-one
        aws_rhel_os_version: rhel-9
        aws_rhel_subnet_id: subnet-0123456789abcdefg
        aws_rhel_security_groups:
          - public-ssh
        aws_rhel_key_name: my-aws-key
        aws_rhel_tags:
          owner: test-user

    - name: Retrieve information on a named VPC created by
            the jce_redhat.cloud_infra.aws_vpc role
      ansible.builtin.include_role:
        name: jce_redhat.cloud_infra.aws_vpc
	tasks_from: role_output.yml
      vars:
        aws_vpc_name: my-test-vpc
        aws_vpc_set_role_output: true

    - name: Get resource IDs from role output
      ansible.builtin.set_fact:
        my_subnet_ids: '{{ _aws_vpc_role_output.subnets | map(attribute="subnet_id") }}'
        
    - name: Create RHEL instance in a random subnet from a named VPC
            created by the jce_redhat.cloud_infra.aws_vpc role
      ansible.builtin.include_role:
        name: jce_redhat.cloud_infra.aws_rhel_instance
      vars:
        aws_rhel_instance_name: test-instance-two
        aws_rhel_subnet_id: '{{ my_subnet_ids | random }}'
        aws_rhel_security_groups:
          - public-ssh
        aws_rhel_key_name: my-aws-key
        aws_rhel_tags:
          owner: test-user
...
```
