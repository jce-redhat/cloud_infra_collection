# cloud\_infra - create simple cloud infrastructure

This Ansible collection includes roles for creating and destroying simple cloud infrastructure.  This collection is primarily meant for standing up quick demo environments, so the roles are not designed for complex cloud infrastructure scenarios.

## Roles

| Role | Description |
|------|-------------|
| [aws\_vpc](roles/aws_vpc/) | Create an AWS VPC with an internet gateway and one or more subnets and security groups |
| [aws\_rhel\_instance](roles/aws_rhel_instance/) | Create a RHEL instance in an existing VPC subnet |

## Playbooks

The following playbooks are included with the collection.

- build-vpc.yml - uses the aws\_vpc role to create a VPC
- teardown-vpc.yml - deletes a VPC that was created by the aws\_vpc role

## License

[MIT No Attribution](LICENSE)
