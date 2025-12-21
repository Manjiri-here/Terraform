terraform
terraform installation
# macOS
> brew tap hashicorp/tap
> brew install hashicorp/tap/terraform

# debian based linux
> wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
> echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
> sudo apt update && sudo apt install terraform

# redhat based linux
> sudo yum install -y yum-utils
> sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
> sudo yum -y install terraform

# windows
> https://releases.hashicorp.com/terraform/1.13.3/terraform_1.13.3_windows_386.zip
verify the installation by checking the version

# get the version
> terraform -version
AWS CLI installation

# linux
> curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
> unzip awscliv2.zip
> sudo ./aws/install

# windows
> msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

# macOS
> curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
> sudo installer -pkg AWSCLIV2.pkg -target /

# or you can use brew
> brew install awscli

# confirm installation
> aws --version

configure AWS CLI:
create a IAM user in AWS management console
give admin privileges
create access key and secret key

# configure aws cli
> aws configure

# verify the aws cli configuration
> aws configure list

# verify if aws cli configured properly
> aws s3 ls
terraform commands
# download the required providers/plugins
> terraform init

# check if the terraform manifest file contains any error
> terraform validate

# fix the manifest file formatting
> terraform fmt

# check the plan
> terraform plan

# apply the plan to create resources
> terraform apply

# destroy the resources
> terraform destroy




**AWS Lambda**

resource "aws_lambda_function" "example" {
  filename         = data.archive_file.example.output_path
  function_name    = "example_lambda_function"
  role             = aws_iam_role.example.arn
  handler          = "index.handler"
  source_code_hash = data.archive_file.example.output_base64sha256

  runtime = "nodejs20.x"

  environment {
    variables = {
      ENVIRONMENT = "production"
      LOG_LEVEL   = "info"
    }
  }

  tags = {
    Environment = "production"
    Application = "example"
  }

**AWS Lambda using module (the below code is to create with local storage and we can also add S3 storage bucket):**

  module "lambda_function" {
  source = "terraform-aws-modules/lambda/aws"

  function_name = "my-lambda1"
  description   = "My awesome lambda function"
  handler       = "index.lambda_handler"
  runtime       = "python3.12"

  source_path = "../src/lambda-function1"

  tags = {
    Name = "my-lambda1"
  }
}
