How to import resources in terraform: If your main.tf file already contains resources then you can create another folder and create main.tf file in it to import below resource:

1. Note down the resource identifier: Like for ec2 I have:  i-07159fd97f818e966

2. Write basic resource block, no need to write full details: Put resource id and name correctly  
provider "aws" {
region = "us-east-1"
}
resource "aws_instance" “terraform-import” {
}

3. Now run-
terraform init
terraform import aws_instance.terraform-import i-07159fd97f818e966

4. % terraform state show aws_instance.terraform-import

5. Now you will see a big output. Copy that and paste it in main.tf file and run “terraform plan”

6. Now run “terraform plan”:

import_folder % terraform plan aws_instance.terraform-import: Refreshing state... [id=i-07159fd97f818e966] 
No changes. Your infrastructure matches the configuration. Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed. 
