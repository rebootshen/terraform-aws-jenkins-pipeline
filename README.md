# terraform-aws-jenkins-pipeline

Terraform and a Jenkins pipeline that deploy a highly available static site on AWS.

## Tech stack

- Terraform  (AWS provider 5.9, region `us-east-1`)
- Jenkins declarative pipeline (`Jenkinsfile`)
- Docker image with Jenkins, Docker, and Terraform (`local-jenkins/`)
- Ubuntu 20.04, Apache, Application Load Balancer, Auto Scaling

## Architecture

Terraform builds one VPC (`10.0.0.0/16`) with two public subnets in `us-east-1a` and `us-east-1b`, an internet gateway, a public route table, and a security group. An Application Load Balancer target group fronts an Auto Scaling group that keeps two Ubuntu instances (minimum 2, maximum 2). Instance user data installs Apache and serves a one-page site that shows the hostname.

Jenkins runs the same Terraform code as a pipeline: init, plan, manual approval, apply, or destroy. AWS keys stay in the Jenkins credential store. They are not committed.

`local-jenkins/` builds a local Jenkins controller so the pipeline can be tried without a shared CI server. The `static` branch holds the sample `index.html`.

## Layout

```text
network/          VPC, subnets, routing, load balancer, security group
compute/          launch template and Auto Scaling group
script/           instance user data
static/           sample static page
local-jenkins/    local Jenkins image and Terraform
main.tf           wires the network and compute modules
providers.tf      AWS provider
backend.tf        S3 remote state
outputs.tf
Jenkinsfile       init, plan, approve, apply, destroy
env.local.example variable names for local AWS credentials
```

## Run

### AWS from the workstation

1. Copy `env.local.example` to `env.local` and export it, or use `~/.aws/credentials`.
2. Create the S3 bucket named in `backend.tf` (see `TF_STATE_BUCKET` in `env.local`) before `terraform init`.
3. From this directory:

```bash
terraform fmt
terraform init
terraform plan
terraform apply
terraform destroy
```

### Local Jenkins

```bash
cd local-jenkins
docker build -t jenkins:terraform .
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

Open `http://localhost:8080`. The initial admin password is printed by:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Install the pipeline, credentials, Terraform, and AnsiColor plugins. Add a Terraform installation under Manage Jenkins, and store `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as Jenkins credentials. The pipeline parameters are `environment`, `autoApprove`, and `destroy`.

## What this demonstrates

- Infrastructure as code split into network and compute modules
- A multi-AZ load-balanced web tier
- Pipeline as code, with a manual approval gate before apply
- Remote state in S3, and a repeatable local Jenkins image

## Secrets

Real AWS keys are not in this repository. Put them in `env.local` (gitignored) or in Jenkins credentials. `local-jenkins/tfplan` is a local plan file and is gitignored.

## License

No license file is included yet.
