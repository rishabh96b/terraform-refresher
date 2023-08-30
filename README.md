# Terraform Notes

I have created these notes as part of my preparation for Hashicorp Terraform Certified Associate Exam (003). These notes can be used to prepare for the certification exam or as a refresher for your terraform knowledge. Feel free to submit a PR if you have any suggestions.

*Table of Contents:*
- [Important Resources](#important-resources)
  * [Terraform Courses](#terraform-courses)
  * [Mock Exams](#mock-exams)
  * [Docs](#docs)
- [Infrastructure As Code(IaC)](#infrastructure-as-codeiac)
- [Core Terraform Workflow](#core-terraform-workflow)
- [Initialise the working directory](#initialise-the-working-directory)
- [Validate](#validate)
- [TERRAFORM PLAN](#terraform-plan)
- [TERRAFORM APPLY](#terraform-apply)
- [TERRAFORM DESTROY](#terraform-destroy)
- [Resource Addressing in Terraform](#resource-addressing-in-terraform)
- [Terraform Providers](#terraform-providers)
- [Terraform STATE](#terraform-state)
- [Provider and Resource Blocks](#provider-and-resource-blocks)
  * [Data Block](#data-block)
  * [Output Block](#output-block)
- [Variables](#variables)
  * [Precedence Order Diagram](#precedence-order-diagram)
- [Provisioners](#provisioners)
- [Terraform State Command](#terraform-state-command)
  * [Points on Terraform State](#points-on-terraform-state)
- [Meta-Arguments](#meta-arguments)
- [Local and Remote Storage](#local-and-remote-storage)
- [Migrating to Terraform Cloud or Terraform Enterprise](#migrating-to-terraform-cloud-or-terraform-enterprise)
- [Embedded Policy As Code: Sentinel](#embedded-policy-as-code-sentinel)
- [Hashicorp Vault](#hashicorp-vault)
- [Terraform Cloud](#terraform-cloud)
  * [Benefits of Terraform Registry and Terraform Cloud Workspaces](#benefits-of-terraform-registry-and-terraform-cloud-workspaces)
- [Important Miscellaneous Points](#important-miscellaneous-points)

## Important Resources

### Terraform Courses
- [FreeCodeCamp (Course By Andrew Brown)](https://www.freecodecamp.org/news/hashicorp-terraform-associate-certification-study-course-pass-the-exam-with-this-free-12-hour-course/) - **FREE**
- [Kodekloud](https://www.udemy.com/course/terraform-for-the-absolute-beginners/) - **PAID**
- [A Cloud Guru](https://www.pluralsight.com/cloud-guru/courses/hashicorp-certified-terraform-associate) - **PAID**

### Mock Exams
- [HashiCorp Certified: Terraform Associate Practice Exam by Bryan Krausen](https://www.udemy.com/course/terraform-associate-practice-exam/) - BEST & PAID
- [ExamPro](https://www.exampro.co/terraform) - 1 FREE Exam
- [Pearson Terraform Associate Exam Cram](https://www.oreilly.com/live-events/hashicorp-terraform-associate-exam-cram/0636920082746/0636920085576/) - Needs Oreilly Subscription, Can be attempted with free trial (no credit card).

### Docs
- https://developer.hashicorp.com/terraform
- [Official Hashicorp Associate Tutorial List](https://developer.hashicorp.com/terraform/tutorials/certification-associate-tutorials-003)

## Infrastructure As Code(IaC)

- Infrastructure is described using a high-level configuration syntax.
- This allows a blueprint of your datacenter to be versioned and treated as you would any other code. Additionally, infrastructure can be shared and re-used.
- Infrastructure as Code almost always uses parallelism to deploy resources faster.
- And depending on the solution being used, it doesn't always have access to the latest features and services available on cloud platforms or other solutions.

## Core Terraform Workflow

WRITE -> PLAN -> APPLY

## Initialise the working directory

**terraform init**

1. Downloads the ancilliary components required for your code to work. Like Providers, modules etc.
2. Sets up back-end for storing Terraform state file.

## Validate

`terraform validate`

The terraform validate command validates the configuration files in a directory, referring only to the configuration and not accessing any remote services such as remote state, provider APIs, etc.

> **INFO:**
> `terraform plan` command runs the validation implicitly.

## TERRAFORM PLAN

`terraform plan -out test`

Allows users to review the action plan before executing anything. It performs the API calls with the specified provider
backend but does not create/modify anything.

Also authenticates with the platform.

## TERRAFORM APPLY

`terraform apply test`

1. Deploy the instructions and statements in the code.
2. Creates the state file `terraform.tfstate`.

> **INFO:**
> When you apply a particular plan generated using `-out` flag, terraform won't ask you for approval.

## TERRAFORM DESTROY

`terraform destroy` / `terraform apply -destroy`

Destroys the provisioned resourced gracefully.

## Resource Addressing in Terraform

**provider**, **resource**, **data**, **module**

Refer to [Resource Addressing](https://developer.hashicorp.com/terraform/cli/state/resource-addressing) for more details.

## Terraform Providers

- Terraform relies on plugins called providers to interact with cloud providers, SaaS providers, and other APIs.
- Providers are distributed separately from Terraform itself, and each provider has its own release cadence and version numbers.
- The [Terraform Registry](https://registry.terraform.io/browse/providers) is the main directory of publicly available Terraform providers, and hosts providers for most major infrastructure platforms.
- Best practice is to specify the version of provider so that tf does not download the latest one when initialising.

## Terraform STATE


- State helps in **RESOURCE TRACKING**!
- Stored into flat files as JSON data.
- The default name of state file is `terraform.tfstate`.
- The default backend of terraform when run locally is local, named as `terraform.tfstate`.

---

## Provider and Resource Blocks

```go
provider "aws" {
		region = "us-east-1"
}
```

```go
resource "aws_instance" "my_instance" {
		ami = ""
		instance_type = "t2.micro"
}
```

### Data Block

TF fetches data from an already existing resource

```go
data "aws_instance" "vm" {
		instance_id = "i-ddddddddddddddd"
}
```

The difference between data and resource block is that data block fetches the information from a preexisting resource and will not create one.

**Resource Address:** data.aws_instance.vm 

### Output Block

```go
output "instance_ip" {
		description = "Instance IP"
		value = resource.aws_instance.my_instance.private_ip
}
```

## Variables

- If the variable stores sensitive value, use `sensitive = true` inside the variable block.
- Add validation to the variable using `validation` argument.

**List Type Variable**

```go
variable "availability_zone_names" {
		type = list(string)
		default = ["us-west-1a", "us-east-1a"]
}
```

> **Highest Precedence is given to the values from the ENV vars & then .tfvars file while setting values of variables in terraform.**
> 

### Precedence Order Diagram

1 is the lowest precedence and 4 is the highest precedence.

| Precedence  | Option                            |
|-------------|-----------------------------------|
| 1           | Environment Variables             |
| 2           | terraform.tfvars file             |
| 3           | *.auto.tfvars (alphabetical order)|
| 4           | -var or -var-file (CLI flags)     |


## Provisioners

Does specified action during creation/destruction of the resource. It will run commands that you’ve specified.

- Each individual resource can have its own provisioner defining the connection method(ssh or WinRM) and the actions/commands or scripts to execute.
- Types: **Creation Time** and **Destroy Time** provisioners.
- Use provisioners **ONLY** if necessary.
- Terraform cannot keep track of provisioners modifications in the state file. If TF tried to do so, it would interrupt the declarative model of TF.
- If the command within the provisioner returns non zero code, it is considered as fail and underlying resource is tainted. i.e marks the resource to be provision again on next run.

```go
resource "null_resource" "dummy" {
		// Creation Time Provisioner
		provisioner "local-exec" {
				command = "echo '0' > status.txt"
		}
		// Destroy Time Provisioner
		provisioner "local-exec" {
				when = destroy
				command = "echo '1' > status.txt"
		}
}
```

To reference the resource name within the same resource block, use `self` to avoid **cyclical dependencies**. Which might refer to the resource which is not yet created.

```go
resource "aws_instance" "analytics-app" {
		ami = ami-sdfgasdfasdf
		instance_type = t2.micro
		subnet_id = aws_subnet.analytics-subnet.id
		associate_public_ip_address = true
		key_name = aws_key_pair.master-key.key_name
		vpc_security_group_ids = [........]
		provisioner "local-exec" {
				command = "aws ec2 wait instance-status-ok --region us-east-1 --instance-ids ${self.id}"
		}
}
```

On **unsuccessful** execution of:

- **Creation Time Provisioner**:
    - Resource gets tainted and TF attempts to re-provision it in next run.
- **Deletion Time Provisioner**:
    - Apply errors and resource is attempted to be destroyed in next run.

## Terraform State Command

### Points on Terraform State

- state file is the mapping between real world resources and terraform configuration.
- resource dependency metadata is also tracked using the state file. So that terraform know to deploy subnet before creating an ec2 instance.
- helps in caching the resource attributes to minimize the API calling.
- TF refreshes the state file prior to any modification operation.

- Command use for manipulate and read tf state file.
- Scenarios:
    - Advance state management.
    - Manually remove resources from tf state file so that it’s not managed by terraform.
    - list the tracked resources.

**Common Commands:**

| Command | Use |
| --- | --- |
| terraform state list | list the resources tracked by tf state file. |
| terraform state rm | delete a resource from terraform state file. |
| terraform state show | show details of a resource tracked in state file. |

---

## Meta-Arguments

Special arguments(or configuration options) that are used with resource blocks and modules to control their behaviour and influence the infrastructure provisioning process.

List of meta arguments:

- `depends_on`:
- `count`:
- `for_each`:
- `provider`:
- `lifecycle`: Some details of the lifecycle of resource behaviour can be controlled using **lifecycle** block within the resource block.
    
    ```go
    resource "docker_container" "my_container" {
    		...
    		lifecycle {
    				create_before_destroy = true
    	}
    }
    
    // create_before_destroy changes the default behaviour of terraform to destroy the resource first
    // and create it if resource modification is not possible using API. With this argument set to true,
    // terraform will create another resource first and then destroy this resource.
    ```
    

## Local and Remote Storage

- By default TF stores the state file locally.
- State locking is enabled by default on local system.
- Remote storage is available by few providers like s3, GC Storage etc.
- Storage like S3 etc also has remote version locking mechanism which prevents parallel writes to the state file.
- For remote backend, we need to add `backend "" {}` block inside `terraform {}` block.

```go
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
  required_version = ">= 0.13"
  backend "s3" {
    profile = "demo"
    region  = "us-east-1"
    key     = "terraform.tfstate"
    bucket  = "<AWS-S3-BUCKET-NAME-GOES-HERE>"
  }
}
```

## Migrating to Terraform Cloud or Terraform Enterprise

This requires migrating the Terraform state files for those resources to one or more Terraform Cloud workspaces. You can perform this migration with either the *Terraform CLI* or the *Terraform Cloud API*.

Refer to [Migration](https://developer.hashicorp.com/terraform/cloud-docs/migrate) docs for more details.

## Embedded Policy As Code: Sentinel

It is a policy as code framework which enables the CIS like rule enforcement like “not allowing access to port 22 in aws instances”, “ensuring every ec2 instance has tag associated with it” etc.

## Hashicorp Vault

- Secrets management software
- Dynamically provision credentials and rotates them for accessing services like AWS instead of using permanent credentials like aws cli access key.
- Encrypts sensitive data **in transit** and **at REST** and provides fine grained ACLs.


## Terraform Cloud

Terraform Cloud is an application that helps teams use Terraform together. 
- Manages Terraform runs in a consistent and reliable environment.
- Easy access to shared state and secret data, access controls for approving changes to infrastructure.
- A private registry for sharing Terraform modules.
- Detailed policy controls for governing the contents of Terraform configurations, and more.

### Benefits of Terraform Registry and Terraform Cloud Workspaces

- Remote Terraform Execution.
- Workspace based model enabling teams to have their own workspaces.
- Integration with VCS like Github and Gitlab.
- Remote state management and CLI execution.
- Private tf registry.
- Cost estimation and sentinel integrations.

Refer to [Terraform Cloud Docs](https://developer.hashicorp.com/terraform/cloud-docs) for more info.

## Important Miscellaneous Points

- Before a `terraform validate` can be run, the directory must be initialized using `terraform init`.
- You have a Terraform configuration file with no defined resources. However, there is a related state file for resources that were created on AWS. What happens when you run a *`terraform apply`?*
    - Terraform will **DESTROY** all the resources to match the current state of the configuration.
- When working with outputs, you need to determine where the value will be coming from and work your way backward from there. For example, if the resource was created inside of a module, then the module will require an output block to export that value. That said, output blocks that are created in a module aren't displayed on the Terraform CLI. Therefore, you need to create an output block in the parent/calling module to output the value while referencing the output in the module. Because of this, the correct answer requires you to create an output in the parent module and reference the output value from the module.
