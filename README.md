# Terraform up & running notes

`The goal of devops is about making software delivery vastly more efficient`

## Infrastructure as code (IaC)

Here's 5 examples of infrastructure as code.

### 1. Ad hoc scripts

Example being PowerShell, Python, Bash

- written in scripting language
- good for one-off tasks
- hard to maintain larger projects

### 2. Configuration management tools

Configures 1 or more VMs

- Agent or agentless
- Master or masterless
- May be idempotence
- scales better

### 3. Server templating tools

Creates a fixed image/snapshot of a system

- docker, packer, vagrant
- immutable (deploy once, redeploy instead of change)
- for example, use packer to create a specific image, deploy it on many servers using ansible

### 4. Orchestration tools

- kubernetes, docker swarm, nomad
- used to monitor, update, scale, load balance servers/services

### 5. Provisioning tools

- used to actually create the virtual machine
- terraform, cloudformation, openstack heat, pulumi
- creates resources using code (iac)

## Benefits of iac

- Faster to deploy code
- Faster to recover from failure
- Faster to deploy servers
- Self-service, users can create servers themselves after approvals
- Speed and safety, automation is fast and consistent
- The code is also documented in detail, usually lacking in manual processes
- Versioncontrolled code makes tracking bugs easy and reverting to a functional state possible
- Validate with tests, catch bugs before they exist
- Reuse modules, save time
- More fun!

## How does Terraform Work

- Written in GO, currently closed-sourced
- Create Terraform configurations (text files), creates infra

1. Have a repository with terraform configuration files
2. commit changes
3. have an action that runs terraform apply
4. infra is created

Terraform does not make the exact same code possible for all cloud providers, since the providers themselves does not provide the same hardware
However you can use the same practices, language and toolset to deploy to many different cloud providers

### procedural approach vs declarative approach

ansible is an example of procedural, it creates resources, without being aware of what it already created

terraform is declarative, meaning you you want to increase the count from 10 to 15, simply change 10 -> 15, in ansible you would change 10 to 5 (to make 5 more).

## Master/agent vs master/agentless

Puppet and chef uses master -> agent -> client approach

Terraform, ansible, cloudformation, openstack heat, and pulumi are all agentless and masterless (mostly)

Terraform utililizes the cloud providers agent

## Popularity of IAC tools

- Ansible and Terraform does have explosive groth periods
- Google trends show terraform as clear winner
- Increase in github stars, new libraries, favor terraform
- Increase in contributors favor ansible

## Maturity

Chef and puppet has been around since 2009 & 2005, they're very mature

Ansible, CloudFormation and Terraform has been around since 2012, 2011 and 2014, somewhat mature by now

Pulimi seems newest and least mature

### Combinations of IAC tools

Combine IAC tools to benifit from their better parts

#### Provisioning plus configuration

1. Use a provisioning tool (like terraform) for new servers
2. Use Ansible to control the OS/Software on the server

This approach is agentless and masterless, no needed infra

#### Provisioning plus server templating

Terraform + Packer

1. Create an image using Packer
2. Deploy the servers with the different images using Terraform

This is agentless, masterless and immutable infrastructure

Drawback is that it's slow, a VM can take long time to build (20-30+m, before provisioning)

#### Provisioning plus server templating plus  orchestration

Terraform, Packer, Docker and Kubernetes

1. Use Packer to create the VM Image that has docker and kubernetes agents installed
2. Use Terraform to deploy many servers (a cluster) that uses this image
3. Configure the rest of your infra with Terraform, subnets, routing tables, databases, loadbalancers
4. When the servers are up, it forms a kubernetes cluster that containerized your application

It's quick since docker builds quick

It adds a lot of complexity, adding cost to managing, takes time to learn all of the IaC systems

## Table 1-4. A comparison of the most common ways to use the most popular IaC tools

|              | Chef  | Puppet | Ansible | Pulumi | CloudFormation | Heat  | Terraform |
|--------------|-------|--------|---------|--------|----------------|-------|-----------|
| Source       | Open  | Open   | Open    | Open   | Closed         | Open  | Closed    |
| Cloud        | All   | All    | All     | All    | AWS            | All   | All       |
| Type         | Config mgmt | Config mgmt | Config mgmt | Provisioning | Provisioning | Provisioning | Provisioning |
| Infra        | Mutable | Mutable | Mutable | Immutable | Immutable | Immutable | Immutable |
| Paradigm     | Procedural | Declarative | Procedural | Declarative | Declarative | Declarative | Declarative |
| Language     | GPL   | DSL    | DSL     | GPL    | DSL            | DSL   | DSL       |
| Master       | Yes   | Yes    | No      | No     | No             | No    | No        |
| Agent        | Yes   | Yes    | No      | No     | No             | No    | No        |
| Paid Service | Optional | Optional | Optional | Must-have | N/A | N/A | Optional |
| Community    | Large | Large  | Huge    | Small  | Small          | Small | Huge      |
| Maturity     | High  | High   | Medium  | Low    | Medium         | Low   | Medium    |

## Terraform

### Variables

#### Input variable

Variables are declared outside of the resources

Variables are accessed using the variable reference syntax:
`var.<VARIABLE_NAME>`

Variables can be populated using the -var option `terraform x -var "<VARIABLE_NAME>=<value>"`

or environment variables as so: `export TF_VAR_<VARIABLE_NAME>=<VALUE>`

alternativly, using the `default` variable parameter.

```tf
variable "NAME" {
    description = "always put a variable description in"
    default = "fallback for variable declaration, if not declared in cli (using -var), or via file using var-file, or via env vars, it fallbacks to default, same as default variable "
    type = "allows to enforce type constraints on variables, supports string, number, bool, list, map, set, object, tuple and any"
    validation = "enables custom validation rules for input, minimum, maximum value on a number, etc"
    sensitive = true # excludes it from logging, use on secrets
}
```

#### Output variables

Supports `description`, `sensitive`, `depends_on`.

```tf
output "public_ip" {
    value = aws_instance.example.public_ip
    description = "The pub ip addr of the webserver"
}
```