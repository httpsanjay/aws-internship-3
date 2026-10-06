# AWS CloudFormation Multi-Tier Capstone

A simple multi-tier AWS environment provisioned entirely using AWS CloudFormation.

This project demonstrates Infrastructure as Code by defining networking, compute, database, permissions, monitoring, backup, and teardown in a CloudFormation template.

## Architecture

```text
                    Internet
                       |
                       |
                +-------------+
                |    EC2      |
                | Web Server  |
                | Public Subnet
                +------+------+
                       |
                       | PostgreSQL :5432
                       |
                +------+------+
                |     RDS     |
                | PostgreSQL  |
                | Private     |
                | Subnets     |
                +-------------+
```

## AWS Resources

The CloudFormation template creates:

- VPC
- Internet Gateway
- Public subnet
- Two private database subnets
- Route table
- EC2 instance
- EC2 security group
- RDS PostgreSQL database
- RDS subnet group
- RDS security group
- IAM role
- IAM instance profile
- CloudWatch CPU alarm

## Multi-Tier Design

The environment is separated into:

### Web / Application Tier

An EC2 instance runs a simple Apache web server in a public subnet.

### Data Tier

PostgreSQL runs on Amazon RDS in private subnets.

The RDS instance does not have a public IP address.

### Security

The database security group only permits PostgreSQL traffic from the EC2 security group.

```text
Internet
    |
    v
EC2
    |
    | 5432
    v
RDS PostgreSQL
```

## Infrastructure as Code

The complete infrastructure is defined in:

```text
template.yaml
```

No resources need to be manually created through the AWS console.

## Prerequisites

Install:

- AWS CLI
- AWS account
- CloudFormation access
- An existing EC2 key pair

Configure AWS CLI:

```bash
aws configure
```

Check the configuration:

```bash
aws sts get-caller-identity
```

## Validate the Template

Run:

```bash
aws cloudformation validate-template \
  --template-body file://template.yaml
```

A successful response confirms that CloudFormation accepts the template.

## Deploy

Create the stack:

```bash
aws cloudformation create-stack \
  --stack-name aws-multi-tier-capstone \
  --template-body file://template.yaml \
  --parameters \
    ParameterKey=KeyPairName,ParameterValue=YOUR_KEY_PAIR \
    ParameterKey=DBUsername,ParameterValue=appuser \
    ParameterKey=DBPassword,ParameterValue=YOUR_PASSWORD
```

Check stack status:

```bash
aws cloudformation describe-stacks \
  --stack-name aws-multi-tier-capstone
```

## Get the Website URL

After deployment:

```bash
aws cloudformation describe-stacks \
  --stack-name aws-multi-tier-capstone \
  --query "Stacks[0].Outputs"
```

Open the EC2 website URL shown in the output.

## Backup

RDS automated backups are enabled.

The database uses:

```text
Backup retention: 7 days
```

The database is also configured with:

```text
DeletionPolicy: Snapshot
```

This creates a final snapshot when the CloudFormation resource is deleted.

## Monitoring

CloudWatch monitors EC2 CPU utilization.

An alarm is created when average CPU utilization remains above 70% for two consecutive five-minute periods.

## Security

The database is not publicly accessible.

The RDS security group allows:

```text
EC2 Security Group
        |
        | TCP 5432
        v
RDS PostgreSQL
```

The EC2 instance uses an IAM role instead of storing AWS access keys on the server.

## Teardown

Delete the CloudFormation stack:

```bash
aws cloudformation delete-stack \
  --stack-name aws-multi-tier-capstone
```

Check deletion:

```bash
aws cloudformation describe-stacks \
  --stack-name aws-multi-tier-capstone
```

The RDS database is configured to create a final snapshot when the resource is removed.

## Project Structure

```text
aws-cloudformation-capstone/
|
├── template.yaml
├── README.md
├── RUNBOOK.md
├── architecture.png
└── .gitignore
```

## What This Project Demonstrates

- Infrastructure as Code
- AWS networking
- VPC and subnet design
- EC2 provisioning
- RDS PostgreSQL
- IAM roles
- Security groups
- CloudWatch monitoring
- Automated database backups
- CloudFormation validation
- Infrastructure teardown

## Author

Sanjay

B.Tech Computer Science Engineering
