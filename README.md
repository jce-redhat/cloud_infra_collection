# cloud\_infra - a collection of Ansible roles for cloud infrastructure

This Ansible collection includes roles for managing cloud infrastructure.

## Roles

| Role | Description |
|------|-------------|
| [aws\_vpc](roles/aws_vpc/) | Create an AWS VPC with an internet gateway and one or more subnets and security groups |

## Playbooks

The following playbooks are included with the collection.

- build-vpc.yml - uses the aws\_vpc role to create a VPC
- teardown-vpc.yml - deletes a VPC that was created by the aws\_vpc role

## License

[MIT No Attribution](LICENSE)
