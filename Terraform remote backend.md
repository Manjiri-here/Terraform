Creating Terraform remote backend:
 On your AWS CLI run below command to create an s3 bucket: 
1. Creating a bucket: % aws s3api create-bucket \
  --bucket my-terraform-state-bucket-08122025 \
  --region us-east-1

{
    "Location": "/my-terraform-state-bucket-08122025"
}

<img width="1550" height="451" alt="image" src="https://github.com/user-attachments/assets/97e465dd-7c5d-4cb0-aae2-7686c5fba172" />


2. Enable versioning: (Purpose: rollback if your state gets corrupted.) Now enable versioning for the above created bucket  aws s3api put-bucket-versioning \   --bucket my-terraform-state-bucket-08122025 \   --versioning-configuration Status=Enabled 
3. Create dyanmodb table for state locking: (stop multiple users from running terraform apply at the same time and corrupting state.)  aws dynamodb create-table \  --table-name terraform-lock-table \   --attribute-definitions AttributeName=LockID,AttributeType=S \   --key-schema AttributeName=LockID,KeyType=HASH \   --billing-mode PAY_PER_REQUEST  Output:  {
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "LockID",
                "AttributeType": "S"
            }
        ],
        "TableName": "terraform-lock-table",
        "KeySchema": [
            {
                "AttributeName": "LockID",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "CREATING",
        "CreationDateTime": "2025-12-08T21:29:59.195000+05:30",
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 0,
            "WriteCapacityUnits": 0
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-east-1:265090223876:table/terraform-lock-table",
        "TableId": "46531706-178e-44fe-9230-2c6fb5b36c52",
        "BillingModeSummary": {
            "BillingMode": "PAY_PER_REQUEST"
        },
        "DeletionProtectionEnabled": false
    }
} 

4. Now add this to your main.tf file:  You can add it even before provider block-  terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket-08122025"  # bucket which you created in previous step
    key            = "demo/terraform.tfstate"                               # key path is just folder-like naming inside S3.It doesn’t have to exist before; S3 will create it.
    region         = "us-east-1"
    dynamodb_table = "terraform-lock-table"                       # table which you created in above step
    encrypt        = true
  }


5. Now you will need to do ‘terraform init’ as you have added new backend block, as below: 
% terraform init
Initializing the backend...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "s3" backend. No existing state was found in the newly
  configured "s3" backend. Do you want to copy this state to the new "s3"
  backend? Enter "yes" to copy and "no" to start with an empty state.

  Enter a value: yes

Releasing state lock. This may take a few moments...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.
Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Using previously-installed hashicorp/aws v6.21.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.  5) Once above step is done you will see the state being backed up in s3
￼
<img width="1792" height="407" alt="Pasted Graphic 1" src="https://github.com/user-attachments/assets/3481e2d8-0ab3-44d5-a710-7daaecfa2483" />

