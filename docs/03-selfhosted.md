# Self-hosted Runners

Using self-hosted runners enables secure access to private cloud resources within
CI/CD pipelines in GitHub Actions.

For example, suppose you have deployed a private API Gateway with multiple AWS
Lambda integrations, accessible only to internal DL systems. GitHub’s default
runners cannot reach the API without exposing a public endpoint. However, by
using a self-hosted runner, the API remains private while still supporting full
CI/CD functionality for building, testing, and deployment.

Additionally, self-hosted runners enhance security by keeping sensitive data
within our infrastructure rather than GitHub’s cloud.

This guide outlines reusable artifacts to enable this capability, along with
recommended hosting patterns (ranked by preference) for a secure and
standardised approach.

## Amazon Web Services (AWS)

:::warning
Running self-hosted runners in AWS will require whitelisting in order for the
runners to communicate with GitHub. Please ensure the domains from the GitHub
guide [here](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#communication-between-self-hosted-runners-and-github)
are allowed by your Firewall/Gateway.
:::

### Deployment

#### Terraform

A private Terraform module [terraform-aws-codebuild](https://github.com/Direct-Line-Group/terraform-aws-codebuild/)
is available in the DL GitHub organisation to support the deployment of AWS
CodeBuild used as a self-hosted runner. To add it to your infrastructure repository
please follow this [example](https://github.com/Direct-Line-Group/terraform-aws-codebuild/tree/main/examples/github-runner).

:::tip
The Terraform module does not define your subnets and security groups. This is
for you as a consumer to manage and apply least privileges for IPs and ports.
:::

### Hosting

#### Option 1 - Automation Account

![image info](assets/self-hosted-option1.png)

- The CodeBuild project(s) are segregated from the application environments in
  a different AWS account and VPC.
- Single or multiple CodeBuild projects can be created based on workload
  CI/CD requirements.
- The CodeBuild project(s) in the automation account is segregated into a
  dedicated private subnets for all projects.
- Security Groups applied to the CodeBuild project(s) allow egress only to the
  workload private subnets on the required ports.
- Security Groups applied to the workload (e.g EC2) account resources allow
  ingress from the CodeBuild private subnets on the required ports.

:::tip
Hosting the self-hosted runners in a separate automation account is the preferred
solution in DL and aligns to the company AWS account strategy. Further
options should only be considered by exception and with the agreement of your
CISO representative.
:::

#### Option 2 - Application Account with Automation Subnet

![image info](assets/self-hosted-option2.png)

- The CodeBuild project(s) are in dedicated subnet(s) in the workload AWS Account.
- Single or multiple CodeBuild projects can be created based on workload
  CI/CD requirements.
- Security Groups applied to the CodeBuild projects allow egress only to the
  workload subnets on the required ports.
- Security Groups applied to the workload (e.g EC2) resources allow ingress from
  the CodeBuild private subnets on the required ports.

:::caution
Option 2 should only be considered by exception and with the agreement of your
CISO representative.
:::

#### Option 3 - Application Account with Application Subnet

![image info](assets/self-hosted-option3.png)

- The CodeBuild project(s) are in hosted in the same subnet(s) as the workload
  resources.
- Single or multiple CodeBuild projects can be created based on workload
  CI/CD requirements.
- Security Groups applied to the CodeBuild projects allow egress on the required
  ports.
- Security Groups applied to the workload (e.g EC2) resources allow ingress on
  the required ports.

:::caution
Option 3 should only be considered by exception and with the agreement of your
CISO representative.
:::

## References

- [Self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- [Private networking](https://docs.github.com/en/actions/using-github-hosted-runners/connecting-to-a-private-network/about-private-networking-with-github-hosted-runners)
- [CodeBuild-hosted GitHub Actions runner](https://docs.aws.amazon.com/codebuild/latest/userguide/action-runner.html)
