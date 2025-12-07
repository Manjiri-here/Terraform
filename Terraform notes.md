Terraform

Terraform interview questions: https://media.licdn.com/dms/document/media/v2/D4E1FAQHjfC1KFn3X-A/feedshare-document-pdf-analyzed/B4EZrJj2OZIoAY-/0/1764318212018?e=1765411200&v=beta&t=1K8VN6LpE4OrpRdPw1jQ0k98dTpdJjA5_4Tsid3WVek

How to import resources in terraform: If your main.tf file already contains resources which are running on AWS then you can create another folder and create main.tf file inside to import below resource.  Import steps are as below:
1. Note down the resource identifier: Like for ec2 I have:  i-07159fd97f818e966

2. Write basic resource block in .tf file (also called stubs), no need to write full details: Only put resource name correctly   provider "aws" {
       region = "us-east-1"
       }
   resource "aws_instance" “terraform-import” {
       }
 3) 
terraform init
terraform import aws_instance.terraform-import i-07159fd97f818e966

Now details will be populated in the .tfstate file once the above step is completed.

4. Now run: terraform state show aws_instance.terraform-import. Also check use of ‘% terraform plan -generate-config-out=generated.tf’ here, looks like we can automatically copy the state file to main.tf file.
5. Now you will see a big output. Copy that and paste it in main.tf file and edit it to keep only the required fields, no need to keep such huge file/info.
6. Now run “terraform plan”: (if you get errors read them carefully and troubleshoot) I had got error because I had kept almost all the fields which we get as output in step 4. To fix the error I kept only minimum fields and the error got fixed. Refer terraform directory under Devops to see an example for this in older ‘terraform_import’  manjiri@Abhis-MacBook-Pro import_folder % terraform plan aws_instance.terraform-import: Refreshing state... [id=i-07159fd97f818e966] No changes. Your infrastructure matches the configuration. Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed. 
— 
Terraform code setup (Industry standard):
✅ Terraform code → GitHub repo
✅ State file → S3 backend with versioning
✅ Locking → DynamoDB table
✅ Apply → Only via CI/CD

—

Here are some key takeaways that might help you avoid the same pitfalls:
1. Always review the plan before applying.
 Reviewing the plan output is essential. It ensures you know exactly what Terraform will modify before it touches your environment.
2. Manage your state responsibly.
 The state file is Terraform’s single source of truth. Always store it remotely (for example, S3 with DynamoDB or Terraform Cloud) and never delete it manually. Treat it as critical infrastructure data.
3. Track all manual changes.
 Manual updates made outside Terraform can introduce drift and inconsistencies. Document such changes, or avoid them by keeping everything as code.
4. Tag your resources. - Thi sis nothing but the tag block tag {} you have inside your resource for eg:

tags = {
    Name        = "payments-api"
    Environment = "prod"
    Owner       = "devops"
    CostCenter  = "fintech-01"
  }
 Consistent tagging improves visibility, cost management, and accountability. It’s a simple step that prevents confusion later.
5. Use variables effectively.
 Avoid hardcoding values. Well-structured variables make your code flexible, reusable, and easier to maintain across environments.


Meta arguments: 

Terraform defines meta-arguments as arguments that can be used with every resource type to change the resource’s behavior. Terraform supports the following meta-arguments:
* depends_on 
* count: count is a Terraform meta-argument that streamlines the process of creating multiple resource instances, eliminating the need to duplicate resource blocks. It can be used with both resource and module blocks. To use the count meta-argument, you need to specify the count argument within a block, which accepts a whole number that indicates the desired number of instances to create. Refer this detailed log- https://spacelift.io/blog/terraform-count
* for_each
* provider
* lifecycle
* provisioner 

To get provider block: https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-build  -> This link is very helpful as you will get tutorial and navigate to tabs on left side for any concerns.

Terraform allows me to build AWS resources in a repeatable, documented way, just like source code or a Jupyter notebook allows you to create a programmatic solution in a repeatable, documented way.
Let's say I have a solution that requires an API Gateway, a few Lambda functions, an S3 bucket, and a DynamoDB table. Between those half-dozen resources there are probably hundreds of settings I can choose.
I can click in the AWS console to create those resources. But when I'm done testing and want to deploy my solution to a production environment... now I have to replicate everything, by hand, to a new environment. And troubleshoot why things won't work. And poke deep into IAM policies. Et cetera, et cetera, et cetera. This is time consuming and error prone. It's a poor use of a developer's time, and something that I don't want to be spending money on. One of the best things about Terraform is that by deploying a Terraform config to both dev and prod, I can know with near-absolute certainty that those environments are identical.
If I put my Terraform code in a git repo, now I can maintain a history of infrastructure changes. I can answer questions like: when did the EC2 instance get changed from a small instance to a big one? Who enabled provisioned concurrency on the DynamoDB table?
By linking git commits back to Jira stories, I can not only answer the 'who' and the 'when' but I can now answer 'why' changes were made.
If a new developer is onboarded to the team and wants to understand how the application works, the Terraform config shows every AWS resource involved in the solution. Oh-- and comments can be added to Terraform just like they can be to source code!
These are just a few of the benefits that Terraform brings to the table.
EDIT: to answer the original question, 'when is Terraform needed,' my answer is: on every single project that deploys to the cloud. From day zero. Even on test projects.

>>> IMP:
1. Develop a landing zone Terraform project to build out an AWS VPC with all the proper subnets (public, private, intra), NAT gateways, etc. Use all the public AWS Terraform modules and customize as necessary.
2. Add a vanilla EKS cluster on top of all that with a minimal config using Fargate.
3. Put all of this in a public Github repo under your account and add all the Github Action bells and whistles for Terraform that put all the shiny green badges on your repo's README.
This should take you about a week at the most, even with minimal Terraform experience. Put a link to this repo on your resume and reference it the first time the interviewer says "Terraform".
Bonus points for contributing to an opensource Terraform project (pick a popular Terraform module and browse through the open issues). Put the merged PR link on your resume.

Terraform steps:

manjiri@Abhis-MacBook-Pro terraform %  terraform init
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 4.16"...
- Installing hashicorp/aws v4.67.0...
- Installed hashicorp/aws v4.67.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

manjiri@Abhis-MacBook-Pro terraform % terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.example will be created
  + resource "aws_s3_bucket" "example" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "my-tf-test-bucket"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + cors_rule (known after apply)

      + grant (known after apply)

      + lifecycle_rule (known after apply)

      + logging (known after apply)

      + object_lock_configuration (known after apply)

      + replication_configuration (known after apply)

      + server_side_encryption_configuration (known after apply)

      + versioning (known after apply)

      + website (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.

——

Do we always need to do terraform init after doing any changes or we can do terraform plan directly?
Answer: 
🔹 terraform init
* Initializes your Terraform working directory.
* Downloads providers (like hashicorp/aws) and modules.
* Creates the hidden .terraform folder and terraform.lock.hcl. (terraform.lock.hcl: It only contains provider dependency integrity.)
👉 You only need to run terraform init:
* Once when you first set up the project.
* Again only if:
    * You add/change a provider (e.g., aws, azurerm, google).
    * You add/change a Terraform module.
    * You delete .terraform/ or move your project.   With terraform.lock.hcl:
* ✅ Everyone gets the exact same provider build ✅ CI/CD gets the exact same provider build ✅ No silent upgrades ✅ No random breakage
* 

🔹 terraform plan
* Reads your existing state + code.
* Shows what changes will happen if you apply.
* Does not re-download providers.
👉 You can run terraform plan directly if you’re only changing resources (like security groups, EC2 configs, outputs, etc.) and not touching providers or modules.

🔹 Example Flow
First time:

terraform init
terraform plan
terraform apply
Later (just editing EC2 size, SG, outputs, etc.):

terraform plan
terraform apply
If you update provider version or add a new provider (say you add random or local provider):

terraform init -upgrade
terraform plan
terraform apply

✅ Summary:
* terraform init = only needed for providers/modules setup.
* terraform plan = safe to run after code changes (most cases).
* You don’t need to init every time.
 Terraform commands:

Terraform Commands Overview
* terraform init → initialize project (downloads providers/modules).
* terraform plan → shows proposed changes.
* terraform apply → applies changes.
* terraform destroy → deletes resources.
* terraform state → interact with Terraform’s state file.
Example: 
terraform state show aws_instance.example
terraform state list

egress → outbound traffic (from your instance to the internet or other resources).
from_port = 0 & to_port = 0 → used as placeholders when the protocol is set to -1.
* Normally, these define a port range.
* Example: from_port = 80, to_port = 80 → only HTTP.
protocol = "-1" → means all protocols (TCP, UDP, ICMP, etc.).
cidr_blocks = ["0.0.0.0/0"] → means "allow traffic to any IPv4 address in the world."  By referring to below picture you will will get idea what above egress block means:

￼

>> terraform.tfstate → current truth of your infra.
>> terraform.tfstate.backup → previous version, kept for safety.


——
How state file is saved remotely on cloud? Answer:
Why save state remotely?
* Collaboration → If multiple engineers are running Terraform, you need a single shared state.
* Locking → Prevents 2 people from applying at the same time (which could corrupt infra).
* Backups → State is safely stored in the cloud.
* Security → No state files with sensitive info on local laptops.

🔹 Common Remote Backends
In big organizations, Terraform state is usually stored in one of these:
* AWS → S3 bucket (with DynamoDB table for locking)
* Azure → Azure Blob Storage
* GCP → Google Cloud Storage
* Terraform Cloud / Enterprise → HashiCorp’s managed backend (very common)

🔹 Example: AWS S3 + DynamoDB (most common in orgs)

terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "ec2/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-locks"   # optional but recommended
    encrypt        = true
  }
}
Steps:
1. Create S3 bucket (for storing state file).
2. Enable versioning on bucket (so old states are recoverable).
3. Create DynamoDB table with LockID as primary key → used for state locking.
4. Add above config to your Terraform code.
5. Run: terraform init  Terraform will migrate local state → S3 backend.

🔹 Example: Terraform Cloud (simpler)

terraform {
  backend "remote" {
    organization = "my-org"

    workspaces {
      name = "my-infra"
    }
  }
}
* No need to manage S3/DynamoDB.
* Handles state, locking, and collaboration for you.

🔹 How big orgs usually do it
* AWS users → S3 + DynamoDB (because they already use AWS infra).
* Multi-cloud / HashiCorp-heavy orgs → Terraform Cloud/Enterprise.
* Azure/GCP users → their native storage solutions.
They also usually:
* Encrypt state at rest (S3 server-side encryption, KMS, etc.).
* Restrict access to only DevOps team via IAM policies.
* Automate Terraform via CI/CD pipelines (Jenkins, GitHub Actions, GitLab CI).

✅ Summary:
* Local state = fine for learning.
* Remote backend = required for teams and production.
* AWS S3 + DynamoDB is the most common setup in real orgs.

—

We can retrieve Ip address or any field using below method- (If you do not know exact field then you can also open state file and then find inside it, but as you will have large number of data inside the tfstate file it is hard to check that file manually) 
manjiri@Abhis-MacBook-Pro terraform % terraform state show aws_instance.first_ec2instance| grep public_ip
    associate_public_ip_address          = true
    public_ip                            = "54.91.139.188"

——

Here are some of the ones that stumped me: (The asnwers for all of them are listed below) 
1) What is the difference between 𝐭𝐞𝐫𝐫𝐚𝐟𝐨𝐫𝐦 𝐢𝐦𝐩𝐨𝐫𝐭 and 𝐭𝐞𝐫𝐫𝐚𝐟𝐨𝐫𝐦 𝐭𝐚𝐢𝐧𝐭?
2) How do you manage secrets in Terraform without hardcoding them?
3) What’s the difference between 𝐜𝐨𝐮𝐧𝐭 and 𝐟𝐨𝐫_𝐞𝐚𝐜𝐡? Give a real-world use case.
4) How do you handle drift detection in Terraform?
5) What is a Terraform remote backend, and why is it important?
6) How do you manage multiple environments (dev, staging, prod) in Terraform?
7) Difference between 𝐥𝐨𝐜𝐚𝐥-𝐞𝐱𝐞𝐜 and 𝐫𝐞𝐦𝐨𝐭𝐞-𝐞𝐱𝐞𝐜 provisioners.
8) How do you safely roll back infrastructure changes after a failed deployment?
9) Explain 𝐭𝐞𝐫𝐫𝐚𝐟𝐨𝐫𝐦 𝐫𝐞𝐟𝐫𝐞𝐬𝐡 vs 𝐭𝐞𝐫𝐫𝐚𝐟𝐨𝐫𝐦 𝐩𝐥𝐚𝐧.
10) How do you write reusable Terraform modules?

Truth is…
Most DevOps engineers underestimate 𝐓𝐞𝐫𝐫𝐚𝐟𝐨𝐫𝐦 𝐟𝐮𝐧𝐝𝐚𝐦𝐞𝐧𝐭𝐚𝐥𝐬.
They focus on writing simple .𝐭𝐟 files but struggle with 𝐬𝐭𝐚𝐭𝐞 𝐦𝐚𝐧𝐚𝐠𝐞𝐦𝐞𝐧𝐭, 𝐦𝐨𝐝𝐮𝐥𝐞𝐬, 𝐚𝐧𝐝 𝐫𝐞𝐚𝐥-𝐰𝐨𝐫𝐥𝐝 𝐜𝐨𝐦𝐩𝐥𝐞𝐱𝐢𝐭𝐢𝐞𝐬.
And interviewers know it.
That’s why companies reject candidates for weak Terraform skills, even if they are Kubernetes or AWS experts.
My advice: Don’t just learn the basics.
Master the internals of Terraform — state files, workspaces, modules, and best practices. These are the silent deal-breakers in high-paying DevOps interviews.

Terraform taint:

Important practical details / gotchas
* It changes state only. The real resource remains until apply runs and actually destroys it.
* If your backend is remote (S3 + DynamoDB locking), terraform taint will update the remote state (it respects locks).
* Destroy may be destructive. For stateful resources (DB, volume), recreate can cause data loss; snapshot before.
* Some resources cannot be cleanly recreated (or recreation is slow/expensive). Know the cost.
* After taint, terraform plan will show a -/+ change (destroy then create).
* terraform apply -replace is often safer and cleaner for single-run replacements (no intermediate state change required).
* You can untaint before apply to cancel.

Compare terraform taint vs terraform import (simple table + explanation)
Feature	terraform taint	terraform import
Purpose	Mark an existing managed resource to be destroyed and recreated	Add an existing unmanaged resource into Terraform state
Operates on	Resource already in Terraform state	Resource that exists in cloud but not in Terraform state
Effect on infra immediately	None (state only)	None (state only)
Typical workflow after	terraform plan → shows destroy/create → terraform apply	Add resource block to .tf, run terraform import, then terraform plan to reconcile and update code
Use case	Repair/replace a bad resource or force recreate	Take ownership of manually-created resources without destroying them
Risk	Can destroy data if resource is stateful	If you don’t code the resource correctly after import, subsequent apply might change or destroy it
Opposite command	terraform untaint	terraform state rm (removes from state), or nothing to reverse import
Short version:
* Taint = replace what you already manage.
* Import = start managing something you previously didn’t.

Example scenarios to make it concrete
Scenario A — corrupted web server config
* You have aws_instance.web managed by Terraform. The instance is flaky.
* Do: terraform taint aws_instance.web → terraform apply
* Result: Terraform destroys the old instance and creates a new one with the same .tf config.
Scenario B — someone created an RDS instance manually and you want to manage it
* You write the aws_db_instance.prod resource block in .tf.
* Run: terraform import aws_db_instance.prod <rds-arn>
* Then run terraform plan and update .tf to match actual attributes. No recreate required unless your code differs.

Alternatives and when to prefer them
* terraform apply -replace="addr" — prefer when you want a single-shot replacement without mutating state beforehand. Cleaner in CI.
* terraform state rm <address> followed by terraform import — use if you want to re-import with different identity, but be careful.
* terraform destroy -target=<address> then terraform apply — explicit but riskier; better to use -replace or taint.

Best practices / safety checklist before you taint
1. Backup: snapshot DBs/volumes if the resource is stateful.
2. Check dependencies: ensure replacement won’t break dependent resources or IPs that must be preserved.
3. Use maintenance window: plan downtime if necessary.
4. Prefer apply -replace in CI: one command, auditable.
5. Document in ticket/PR why you tainted a resource.

Final one-line rule to remember
* taint = “make Terraform replace this managed resource on next apply.”
* import = “tell Terraform about an existing resource so it starts managing it.”

when Terraform recreates a resource after taint, it does NOT save or preserve any data on that resource unless you explicitly designed it that way.
If the resource is destroyed, everything ON that resource is gone:
* Files on the disk
* Database data
* App uploads
* Logs stored locally
* Any runtime state
Terraform does zero automatic backup for you. It only follows the destroy → create logic.

What actually happens with taint (step-by-step, no myths)
When you run:

terraform taint aws_instance.web
terraform apply
Terraform does this:
1. DESTROY the old resource
2. CREATE a brand-new resource
3. Attach it to the same Terraform code
It does NOT:
* Copy files
* Migrate data
* Backup volumes
* Preserve runtime state
* Keep old disks (unless you explicitly told it to)
So yes — data loss is the default behavior.

When data is LOST vs SAVED (critical distinction)
❌ Data is LOST when:
* Data is stored on:
    * EC2 root disk
    * Local disk
    * Ephemeral storage
* And you taint:
    * aws_instance
    * aws_db_instance (without snapshots)
    * Any VM with only local storage
Example:
* You stored MySQL data inside /var/lib/mysql on EC2 root volume
* You taint the instance
* Apply runs ✅ Instance recreated ❌ Database permanently deleted

✅ Data SURVIVES when:
Only if you have separated storage from compute:
Setup	Data survives taint?	Why
EC2 + separate EBS volume (not set to delete on terminate)	✅ Yes	Volume is not destroyed
RDS with automated snapshots	✅ Yes	Snapshot restores data
S3 storage	✅ Yes	EC2 replacement doesn’t touch S3
EFS	✅ Yes	Network filesystem
This is the correct safe pattern (production-grade)
Never store important data directly on a resource you might taint.
Correct design:
* EC2 → stateless
* Data → EBS / RDS / S3 / EFS
* Backups → Snapshots + point-in-time restore
Then taint becomes safe.

——

Terraform backs up the state file because losing it can completely screw your infrastructure control. And yes—the backup can save you if the active state file is deleted, but only in some cases. Here’s the clean truth.
I’ll reference this once: Terraform.

1️⃣ What is the Terraform state backup?
When you run:

terraform apply
Terraform automatically creates:

terraform.tfstate
terraform.tfstate.backup
* terraform.tfstate → current truth
* terraform.tfstate.backup → previous truth (1 version old)
That’s it. Only ONE backup. Not history. Not versioned.

2️⃣ Why does Terraform even create a backup?
Because state corruption or deletion is catastrophic. The state tracks:
* What resources exist
* Their real AWS IDs
* Dependencies between them
Without state:
* Terraform forgets everything
* It will try to recreate resources that already exist
* This leads to:
    * Duplicate EC2
    * Broken networking
    * Data loss
    * Double billing
The backup exists for human mistakes and crashes, not luxury.

3️⃣ If the state file is deleted accidentally — does backup help?
✅ YES — if this is your setup:
* You’re using local backend
* terraform.tfstate.backup still exists
You can recover instantly:

mv terraform.tfstate.backup terraform.tfstate
Terraform is back in business.

❌ NO — if this is your setup:
* You use remote backend (S3, Terraform Cloud, etc.)
* Someone deletes the remote state bucket/object
* The local .backup is outdated or unrelated
Now you’re screwed unless:
* S3 versioning is enabled
* Or you exported state earlier

4️⃣ Why backup is NOT enough in real projects
Because:
* It keeps only 1 old copy
* If corruption happens twice → you lose both
* It does NOT protect against remote deletion
* It does NOT help teams
That’s why industry standard is:
✅ Remote backend ✅ S3 versioning ✅ DynamoDB locking
Example:

backend "s3" {
  bucket         = "terraform-state-prod"
  key            = "prod/terraform.tfstate"
  region         = "ap-south-1"
  dynamodb_table = "terraform-locks"
  encrypt        = true
}
Now you get:
* Infinite versions
* Locked updates
* Team safety
* Disaster recovery

5️⃣ Brutal truth you need to remember
* State file = brain of Terraform
* Backup = last memory only
* No state = Terraform becomes blind and destructive
* Local state is for learning only
* Production without remote backend = rookie mistake

6️⃣ Final direct answer to your question
“And in case state file is deleted accidentally will this file come in use?”
✅ Yes — ONLY if you are using local state and the backup still exists. ❌ No — if you’re using remote backend and versioning is OFF.


—

How do you manage secrets in Terraform without hardcoding them?
The core rule
Never put passwords, tokens, or keys directly inside .tf files. If you do, you’ve already failed security.

✅ Correct ways to handle secrets
✅ Option 1: Environment Variables (Most Common)
Instead of:

password = "mypassword"
You do:

export TF_VAR_db_password="mypassword"
And in Terraform:

variable "db_password" {}
Terraform automatically injects it.
✔ Secure ✔ Works in CI/CD ✔ No secrets in Git

✅ Option 2: Secret Managers (Best for Production)
Use:
* AWS Secrets Manager
* HashiCorp Vault
* SSM Parameter Store
Terraform reads the secret, doesn’t store it.

✅ Option 3: .tfvars file (Only for local testing)

db_password = "mypassword"
Then:

terraform apply -var-file="secrets.tfvars"
And .gitignore that file.

❌ Wrong approach:

password = "admin123"
This gets leaked to:
* GitHub
* Logs
* State files
* CI pipelines

3️⃣ Difference between count and for_each
Both are for creating multiple resources, but they behave very differently.

✅ count → Number-based (Indexed)

resource "aws_instance" "web" {
  count = 3
  instance_type = "t3.micro"
}
Creates:

web[0], web[1], web[2]
❌ Why count is risky
If you delete the middle one: Terraform RENUMBERS everything → causes unexpected destruction.

✅ for_each → Key-based (Stable)

resource "aws_instance" "web" {
  for_each = {
    dev  = "t3.micro"
    prod = "t3.large"
  }

  instance_type = each.value
}
Creates:

web["dev"], web["prod"]
✔ No reordering problems ✔ Safe for production ✔ Used with maps & sets

✅ When to use what
Use Case	Use
Fixed number like “3 identical servers”	count
Environments, users, named resources	for_each
4️⃣ How do you handle drift detection in Terraform?
Drift = real infrastructure changed outside Terraform.
Example: You change EC2 instance type manually in AWS Console → Terraform doesn’t know.

✅ How to detect drift
✅ terraform plan
It compares:
* Terraform state
* Real cloud
* Desired code
And shows:

~ instance_type: t2.micro → t3.micro

✅ terraform refresh (now mostly auto)
It updates state from real infrastructure. Does NOT change infra.

✅ Best Practice
* Never change infra manually
* Run terraform plan before every apply
* Detect drift in CI (dangerous drift = blocked)


terraform refresh — WITH REAL EXAMPLES
What terraform refresh actually does (simple)
It does NOT change your cloud. It only updates Terraform’s state file to match what is actually running right now.
Think of it as:
“Terraform, go look at AWS and update your memory.”

✅ Real Example: EC2 drift
Step 1 — Terraform created an EC2

resource "aws_instance" "web" {
  instance_type = "t2.micro"
}
Terraform state says:

instance_type = t2.micro

Step 2 — Someone manually changes EC2 in AWS Console
They change:

t2.micro → t3.micro
Now:
* Terraform state ❌ WRONG
* Actual AWS ✅ CORRECT

Step 3 — You run:

terraform refresh
Terraform now: ✅ Reads real AWS ✅ Updates state to:

instance_type = t3.micro
But: ❌ It does NOT change the server ❌ It does NOT apply your .tf file

Step 4 — Now run:

terraform plan
Terraform says:

Terraform wants to change instance_type back to t2.micro
This is how you detect drift safely.

✅ Key rule to remember
Command	Changes Cloud?	Changes State?
refresh	❌ No	✅ Yes
plan	❌ No	❌ No
apply	✅ Yes	✅ Yes

5️⃣ What is a Terraform Remote Backend & Why Important?
In simple words:
Remote backend = where Terraform stores the state file safely.

❌ Local backend (default)
State stored on:

terraform.tfstate (on your laptop)
Problems:
* Team cannot collaborate
* No locking
* Easy to corrupt
* If laptop dies → infra unmanaged

✅ Remote backend (Industry standard)
State stored in:
* S3 bucket
* Locking via DynamoDB
Benefits:
* Team sharing
* State locking (no double apply)
* Encrypted
* Versioned
* Safe for CI/CD

6️⃣ How do you manage multiple environments (dev, stage, prod)?
You have three correct industry methods:

✅ Method 1: Separate Folders (Most Common)

envs/
  dev/
  staging/
  prod/
Each has:
* Its own backend
* Its own tfvars
* Its own state

✅ Method 2: Workspaces (Less preferred)

terraform workspace new dev
terraform workspace new prod
Same code → separate state files.
⚠ Risky in large teams ⚠ Easy to destroy wrong env

✅ Method 3: Git Branch per Environment
Used in strict compliance orgs.

✅ Best answer in interviews:
“We use separate folders with separate remote backends per environment.”

Separate Folders for dev / staging / prod (Industry Standard)
This is the most used setup in real companies.

✅ Folder Structure

terraform/
  modules/
    ec2/
      main.tf
      variables.tf

  envs/
    dev/
      main.tf
      backend.tf
      terraform.tfvars

    staging/
      main.tf
      backend.tf
      terraform.tfvars

    prod/
      main.tf
      backend.tf
      terraform.tfvars

✅ Does each env have same or different main.tf?
✅ SAME LOGIC (almost always)
Dev, staging, prod use the same module code.
Example envs/dev/main.tf

module "ec2" {
  source = "../../modules/ec2"
  instance_type = "t3.micro"
}
Example envs/prod/main.tf

module "ec2" {
  source = "../../modules/ec2"
  instance_type = "t3.large"
}
Same module ✅ Different size ✅ Different state ✅

✅ Each environment has its OWN state file
dev backend:

bucket = "company-terraform-dev-state"
key    = "ec2/terraform.tfstate"
prod backend:

bucket = "company-terraform-prod-state"
key    = "ec2/terraform.tfstate"
✅ Dev mistakes never touch prod ✅ Separate locking ✅ Separate permissions

3️⃣ Git Branch per Environment (Simple Explanation)
This is less common but still used in strict orgs.

✅ Branch Model

main     → production
staging  → testing
dev      → development

✅ Example Workflow
You change Terraform in dev branch:

git checkout dev
git commit -m "Add new EC2"
Then you promote:

dev → staging → main
Each environment deploys from its own branch.

✅ Why some companies do NOT like this
* Easy to forget to merge
* Hard to track infra parity
* More Git conflicts
* Slower releases

✅ Why folder-based environments are preferred
Reason	Folder Approach
Safety	✅ High
CI/CD	✅ Easy
Auditing	✅ Clear
Human errors	✅ Fewer


7️⃣ Difference between local-exec and remote-exec
Provisioners run after resource creation.

✅ local-exec → Runs on YOUR MACHINE / CI

provisioner "local-exec" {
  command = "echo Server created"
}
Used for:
* Alerts
* Slack notifications
* Trigger scripts

✅ remote-exec → Runs INSIDE THE EC2

provisioner "remote-exec" {
  inline = [
    "sudo apt install nginx -y",
    "sudo systemctl start nginx"
  ]
}
Used for:
* Bootstrapping
* Installing software

❌ Provisioners are NOT recommended for production. ✅ Use:
* Ansible
* User Data
* Packer

Simple definition
A provisioner runs commands after a resource is created.
Terraform creates the server → then provisioner installs software.

✅ Example: Installing Nginx using remote-exec

resource "aws_instance" "web" {
  ami = "ami-123"
  instance_type = "t3.micro"

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install nginx -y",
      "sudo systemctl start nginx"
    ]
  }
}
What happens:
1. Terraform creates EC2 ✅
2. SSH connects ✅
3. Commands run inside EC2 ✅
4. Nginx is installed ✅

✅ local-exec example
Runs on your laptop or CI machine:

provisioner "local-exec" {
  command = "echo EC2 deployed"
}
Used for:
* Slack alerts
* Email notifications
* Triggering scripts

❌ Why provisioners are DISCOURAGED in production
Problem	Why it’s bad
Not idempotent	Can’t safely re-run
Fails easily	SSH issues
No rollback	Half-installed servers
Mixing tools	Infra + config mixed
✅ What companies use instead
* user_data
* Ansible
* Packer
Terraform should: ✅ Create infra ❌ Not configure software deeply



8️⃣ How do you safely roll back after failed deployment?
Terraform does not have a “rollback” button.

✅ Safe Rollback Approach
✅ Step 1: Revert Git Code

git revert <bad-commit>
✅ Step 2: Run Terraform Again

terraform plan
terraform apply
Terraform sees previous config → restores it.

✅ For Databases
* Use:
    * Snapshots
    * Point-in-time recovery
Terraform will not automatically restore data.

9️⃣ terraform refresh vs terraform plan
✅ terraform refresh
* Updates state from real cloud
* Does NOT change infrastructure
* Just syncs numbers
Think:
“What does my infra look like right now?”

✅ terraform plan
* Compares:
    * Code
    * State
    * Real infra
* Shows exactly what WILL CHANGE
Think:
“What will Terraform do if I apply right now?”

✅ Real use case
You manually resized EC2:
* refresh → updates state
* plan → shows mismatch vs .tf

🔟 How do you write reusable Terraform modules?
A module is just:
A folder of Terraform code that takes inputs and produces outputs.

✅ Example Module Structure

modules/ec2/
  main.tf
  variables.tf
  outputs.tf
variables.tf

variable "instance_type" {}
main.tf

resource "aws_instance" "this" {
  instance_type = var.instance_type
}

✅ Using that module

module "web" {
  source = "./modules/ec2"
  instance_type = "t3.micro"
}

✅ Why modules matter
* Reuse code
* Enforce standards
* Fewer mistakes
* Faster deployments
Every real company uses modules.

🔥 Final One-Page Mental Mapping
Concept	Purpose
Secrets	Never hardcode, use env or vault
count	index-based scaling
for_each	key-based safe scaling
Drift	Detect via plan
Remote Backend	Shared, locked state
Environments	Separate folders & backends
local-exec	Runs on your machine
remote-exec	Runs inside server
Rollback	Git revert + apply
Refresh	Sync state
Plan	Predict changes
Modules	Reusable infra blocks

——

Terraform drift 
Terraform Drift happens when you create infra with Terraform but some manual changes added later which are not present in statefile this is called Terraform Drift. 
so to resolve this issue we use below steps:
- First check Terraform Plan it will highlight the drift
Or You can use 
driftctl scan
- if you want to keep changes and update in state file then use "update" command
E.X.
update instance_type in .tf file to t2.small
- if you don't want to do any changes then reapply use
Terraform apply

This question was asked to me in recent HCL interview
Note: this is also considered as answer for statefile corruption and apart from drift there is another solution also i.e. terraformer command I don't know much about it so unable to explain


What is Terraform and how it works? 
-> Terraform is a IAC tool that lets you write the code for the infra or resources you want to create on cloud. When you write the code you then run the terraform commands and terraform willl then create the infra on cloud. Terraform does this using state file. State file stores all the data of all infra managed through Terraform and Terraform uses the state file to compare with state file with actual infra on cloud to tell us what is going to be created or deleted according to the code /config you have in this file.

￼

A devops engineer manually created infra on AWS and now there is a requirement to use terraform to manage it. How would ou import these resources in terraform code?
-> For infra to be managed thorough Terraform it has to be present in state file.SO we will start by creating the configuration, we will create the code (say resource block for particular instance) specifying instance type, the AMI used and other things. Once you write the code for the code to be configured inside state file we will run terraform import command.
We will repeat this step for each instance we want to be managed by Terraform.

You have multiple env. for your applications dev, stage, prod for your application and you want to use the same code for all of these env. How can you do that.
-> We can do these using either Terraform modules or Terraform workspaces
Terraform modules are code templates for infra components, you define it once and use them with different configurations for various env by passing in different parameters or variables.
Using workspaces you will have different state files for different env using the same code. Each workspace maintains its won state file allowing you to work on different env concurrently without interfering with each other. For workspaces you can refer: https://medium.com/capital-one-tech/deploying-multiple-environments-with-terraform-kubernetes-7b7f389e622  Suppose you want to manage both a staging and production environment for the same AWS infrastructure. Without workspaces, you might need to copy your code into different folders—one for each environment—which quickly becomes messy.
With workspaces:
1. You keep a single Terraform codebase for your infrastructure (for example, main.tf).
2. You create separate workspaces:
terraform workspace new staging
terraform workspace new prod
3. Each workspace now maintains its own state:
    * When in the staging workspace, changes only affect the staging environment and its resources.
    * When in the prod workspace, changes affect production resources only.
4. You can switch between environments with: terraform workspace select staging terraform workspace select prod
5. and perform plan/apply tasks for the chosen environment.
     How This Looks in Real Life (Production)
 You use terraform.workspace in your variable files or in resource blocks to customize names or settings per environment, ensuring resources don’t clash between environments.
resource "aws_s3_bucket" "example" {
  bucket = "myapp-bucket-${terraform.workspace}"
}
When in the staging workspace, bucket will be myapp-bucket-staging. In prod, it will be myapp-bucket-prod.

Benefits in Production
* Isolates environments: No accidental changes to production when you’re working in dev or staging.
* Reduces code duplication: Just one codebase to maintain.
* Safe experimentation: Test changes in a workspace before moving to production.
  DRY Principle (Don't Repeat Yourself)
* Meaning: The DRY principle stands for "Don't Repeat Yourself" and means that each piece of logic, knowledge, or data should exist in only one place in your code or system.
* Why it matters: If you avoid duplicating code, you only need to make changes in one place, which reduces errors, speeds up updates, and keeps your code clean and consistent.
* DevOps connection: In DevOps, this could mean using scripts, templates, or modules that are reused for server setup, infrastructure provisioning, or CI/CD jobs instead of copying the same settings everywhere.
Simple analogy: If you need the same phone number in five places in your application, the DRY way is to write it once and reference it, not write it five times.

Mutability & Immutability (in DevOps & Software)
* Mutable: Something that can change after it is created.
    * In DevOps, mutable infrastructure means servers or resources are updated or modified in-place (e.g., patching a running server, changing its config manually).
    * In software, mutable means a variable or object’s value can be changed.
* Immutable: Something that cannot change after it is created.
    * In DevOps, immutable infrastructure means instead of updating a running server, you create a new one with the updates and replace the old one (e.g., build a new machine image, deploy, then delete the old). This helps guarantee consistency, reduces unexpected differences, and simplifies troubleshooting.
    * In software, immutable data means once a value is set, it can’t be changed—if you want something different, you create a new value/object.

What is Terraform state file and why is it important?
It had info of resources managed through Terraform. It is is JSON file . Using state file Terraform compares what you already have on cloud and then it will actually create or delete stuff.

A Junior devops egg. Accidentally deleted the state file , what steps should we take to resolve this?
-> 1) If available, restore state file from backup.
2. If there is no backup you need to manually create the state file you can do it using Terraform state command. You will check what are the resources on the cloud an using Terraform import command you will create every single resource into state file to recreate the state file

What are some best practices to manage Terraform state file?
-> 1) Remote storage- store the state file remotely (eg. AWS S3) for safety, collaboration and version control.
2. State locking: Enable state locking to prevent conflicts in concurrent operations.
3. Acces controls: Limit access to state file to authorised users and services, so accidental deletion and corruption will be limited.
4. Automated backups: Make sure to setup automated backups of state file.
5. Env separation- Maintain separate state file for each env or use Terraform workspace to manage multiple files.


Your team is adopting a multicloud strategy and you need to manage resources on both AWS an Azure using Terraform. So how do you structure your TF code to handle this?
-> Terraform is cloud agnostic- means it can work on different cloud at once 
1. You will first write providers. 2) Then you will write code for different resources you want to create


There are some  bash scripts that you want to run after creating your resources with Terraform so how would you achieve this.
-> You can do this using provisioners. There are 5 different types of provisioners in Terraform. For us to run base scripts in TF we can use local-exec and remote-exec provisioner

Your company is looking ways to enable HA . How can you perform blue green deployments using Terraform ?
-> Blue green deployment is a strategy where you have two identical environments a blue one and green one. Terraform facilitates (helps in this) by defining two sets of infra resources with slight variations
What we do is: Create new env alongside with the existing one which is going to be the green one And you will test if everything is going well in the new env , if everything is working properly then you can switch the traffic either using LB or using DNS records. For more details refer: https://developer.hashicorp.com/terraform/tutorials/aws/blue-green-canary-tests-deployments

Your company wants to automate Terraform through CICD pipelines. How can you integrate Terraform with CICD pipeline?
1. First you need to push the code (main.tf) to GitHub repo/any version control system.
2. Create a CICD pipeline to init, plan and apply the terraform code
3. For apply you can do that step manually so that it will ask for approval , you can also setup approval for deletion as well.
Cloud Camp also has a separate video on that- check that out

Describe how you can use Terraform with Infra deployment tools like Ansible and Chef.
-> Ansible and chef can handle tasks such as installing s/w, configuring servers, managing services while Terraform focuses on infra provisioning and orchestration. Mainly it handles steps after the infra is provisioned.
Code camp also has a video explaining how they both are different.

1) local-exec Provisioner
What it does (simple):
Runs a command on YOUR machine (or the CI server), NOT inside the cloud instance.
Think:
“Terraform created something → now run a local script.”

✅ Example

resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = "t3.micro"

  provisioner "local-exec" {
    command = "echo EC2 created successfully"
  }
}
This runs on:
* Your laptop ❌ not on EC2
* CI/CD server ❌ not on EC2

✅ Real-world uses
* Send Slack alerts
* Trigger shell scripts
* Update DNS
* Run smoke tests
* Call APIs

❌ Do NOT use for:
* Installing nginx
* Configuring servers
* Setting up databases
That belongs inside the server, not on your laptop.

✅ 2) remote-exec Provisioner
What it does (simple):
Runs commands INSIDE the EC2/server via SSH or WinRM.
Think:
“Terraform created the server → now log into it → run commands.”

✅ Example (Install Nginx)

resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = "t3.micro"

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("key.pem")
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install nginx -y",
      "sudo systemctl start nginx"
    ]
  }
}

✅ Real-world uses
* Install nginx
* Setup application
* Add users
* Run scripts

❌ Why companies avoid it in production
* SSH failures break Terraform
* Not idempotent (re-running causes problems)
* No rollback
* Mixing infra + config = bad design

✅ 3) file Provisioner
What it does (simple):
Copies files from your machine → into the server.
Think:
“Send this file into the new EC2.”

✅ Example (copy app config)

provisioner "file" {
  source      = "app.conf"
  destination = "/home/ubuntu/app.conf"
}
Terraform will: ✅ Copy from your laptop ✅ Paste into the EC2

✅ Real-world uses
* Copy scripts
* Copy SSL certs
* Place config files

❌ These are NOT real Terraform provisioners anymore
Thing	Status
Chef	❌ Deprecated
Puppet	❌ Deprecated
Salt	❌ Deprecated
Terraform removed these because:
* They made infra unpredictable
* They caused state drift
* They broke idempotency

✅ Important Concept: null_resource with Provisioners
Terraform allows provisioners only inside a resource, so sometimes we use this fake resource:

resource "null_resource" "setup" {
  provisioner "local-exec" {
    command = "echo Hello"
  }
}
Used when:
* You don’t want to attach provisioners directly to AWS resources

✅ Summary Table (Clean Memory Map)
Provisioner	Runs Where	Used For
local-exec	Your laptop / CI	Scripts, alerts, API calls
remote-exec	Inside EC2	Install software
file	Moves file into EC2	Configs, scripts
✅ The hard reality (important for interviews)
* Provisioners are a last resort
* Production-grade alternatives:
    * user_data
    * Ansible
    * Packer
* Terraform’s job = create infra, not configure apps

✅ Final Straight Answer
Terraform has only THREE real provisioners:
* local-exec
* remote-exec
* file
Anything else is either deprecated or wrong.


Your infra contains db passwords and other sensitive info. How can you manage secrets and sensitive data in Terraform?
-> Best practices: Never hardcode secrets in Terraform code.
Storing secrets outside of version controlled files , using tools like Hashocprp vault or cloud specific secret management tools.
Utilising terraform input variables or env variables to pass sensitive values securely during runtime.

You have a RDS database and an EC2 instance. EC2 should be created before RDS. How can you specify dependencies between resources in terraform.
-> Using depends_on (it is a meta argument) attribute within resource blocks. By including this attribute you define an explicit ordering or resource creation and ensure that one resource is created before another. This helps manage dependencies when one resource relies on the existence or configuration of another resource.

You have 20 servers created through Terraform but you want to delete one of them. Is it possible to delete a single resource out of multiple resources using Terraform?
-> Yes it is. We can use the terraform destroy -target command followed by the resource type and name to destroy a specific resource. 
For ex:
terraform destroy -target=aws_instance.my_instance

Here only the instance named my_instance would be destroyed leaving other resources intact. Link to refer: https://developer.hashicorp.com/terraform/cli/commands/destroy

What are advantages of using Terraform count feature over resource duplication?
-> Count meta argument is used to define how many resources you want to create. Other way to do it is writing the code 4 times.
Refer: https://developer.hashicorp.com/terraform/language/meta-arguments

What is Terraform’s module registry and how can you leverage it?
-> It is central registry where you will find the code rather than writing it yourself(You can share and refuse it). Suppose you want to create EKS cluster than rather than writing code you can use module on registry.  

How can you implement automated testing for Terraform code?
-> By default terraform will give you terraform validate and terraform format command which will check syntax and format the code according TF best practice.
Other than this there are aslo tools like- Terratest, tflint
Refer: https://medium.com/contino-engineering/terraform-infrastructure-as-code-testing-best-practice-unit-tests-bdd-end-to-end-scenario-c30d5a6921d

You are tasked with migrating your existing infra from terraform version 1.7 to 1.8 so what kind of considerations and steps would you take?
-> 1) Review upgrade guide to understand changes , deprecations and new version.
2. Upgrade config file according to changes and next syntax.
3. Address any breaking changes introduces in new TF env.
4. Ensure through non-production testing before moving to prod.
5. Document any changes and update about changes and knowledge to team members.

—

1. What is Terraform and why is it used? 
Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp that allows for declarative management of cloud and on-premise resources. 
2. How do Terraform providers work? 
Providers act as a bridge between Terraform and cloud services, handling API interactions and exposing resources. 
3. What is a Terraform module? 
Modules are reusable configurations that group resources together for better structure and reusability. 
4. How does Terraform state work? 
Terraform state keeps track of resources that have been created, allowing Terraform to know what to create, update, or destroy. 
5. What are workspaces in Terraform? 
Workspaces allow multiple state files for the same configuration, which is useful for managing environments like dev, staging, and prod. 
6. Can you explain the use of the lifecycle block? 
The lifecycle block lets you control resource creation, update, and deletion behavior, including prevent_destroy or ignore_changes. 
7. How do you manage sensitive data in Terraform? 
Sensitive variables can be marked as sensitive, use secret managers, and avoid storing secrets in plain text in configuration files. 
8. What is the difference between Terraform plan and apply? 
terraform plan shows the changes to be applied to the infrastructure, while terraform apply executes these changes. 
9. What is the Terraform registry? 
The Terraform registry is a public repository for sharing and discovering Terraform providers and modules. 
10. What is a data source in Terraform? 
A data source allows you to fetch data about resources defined outside of Terraform or created by another configuration. 
11. What is the count meta-argument? 
The count meta-argument creates multiple instances of a resource or module. 12. How do you manage provider versions? 
You manage provider versions using the required_providers block, specifying a version constraint. 
13. What is the purpose of the .terraform.lock.hcl file? 
This file locks provider versions to ensure that all team members use the same provider versions. 
14. How do you provision resources in a specific order? 
Terraform automatically handles dependencies. If a resource depends on another, Terraform will provision it in the correct order. 
15. What is the local value? 
A local value assigns a name to an expression or a value for use within a configuration.
16. What is a for_each meta-argument? 
The for_each meta-argument iterates over a map or a set of strings to create multiple instances of a resource. 
17. What is a provider block in Terraform? 
A provider block configures the provider, like AWS or Azure, with credentials and settings. 18. What is a remote backend? 
A remote backend stores the state file remotely, enabling team collaboration and state locking. 
19. How do you validate a Terraform configuration? 
You validate a configuration using terraform validate to check for syntax errors and consistency. 
20. How do you import existing infrastructure into Terraform? 
You import existing infrastructure using the terraform import command, which adds it to the state file. 
21. What are dynamic blocks? 
Dynamic blocks allow you to generate nested configuration blocks dynamically based on a collection of values. 
22. What is Terraform cloud? 
Terraform Cloud is a platform for managing Terraform workflows, providing remote state, collaboration, and policy enforcement. 
23. What is the purpose of the terraform init command? 
terraform init initializes the working directory by downloading providers and setting up the backend. 
24. How do you destroy infrastructure with Terraform? 
You destroy infrastructure using terraform destroy, which deletes all resources in the state file. 
25. What is terraform console? 
terraform console is a command-line utility for evaluating expressions and testing logic within the configuration. 
26. How do you manage dependencies between modules? 
Modules can depend on each other by passing outputs from one module as inputs to another. 
27. What is the use of terragrunt? 
terragrunt is a wrapper for Terraform that helps manage dependencies between modules and keeps configurations DRY (Don't Repeat Yourself). 
28. What is the tflint tool? 
tflint is a linter for Terraform that checks for syntax errors, best practices, and security issues. 
29. How do you create a custom Terraform provider? 
You create a custom provider by writing code in Go that interacts with an API and defining resources and data sources. 
30. What is taint in Terraform? 
terraform taint marks a resource for forced destruction and recreation on the next apply.
31. How do you manage infrastructure as a team? 
By using a remote backend, shared modules, version control, and CI/CD pipelines. 32. What is Terraform's backend block? 
The backend block configures how and where Terraform stores the state file. 33. How do you manage different environments with Terraform? 
By using workspaces, separate folders, or a single repository with different configuration files. 
34. What are provisioners? 
Provisioners are used to execute scripts on a local or remote machine after a resource has been created. 
35. What is a null resource? 
A null resource allows you to run provisioners or trigger other actions without creating a physical resource. 
36. How do you manage provider aliases? 
You use provider aliases to configure multiple instances of the same provider with different credentials. 
37. How do you share modules privately? 
You can share modules privately using a private Terraform registry or by referencing modules from a Git repository. 
38. What is the depends_on meta-argument? 
depends_on explicitly defines a dependency between resources that Terraform cannot infer automatically. 
39. What is terraform fmt? 
terraform fmt is a command that automatically formats Terraform code to a standard style. 
40. How do you handle secrets with a remote state? 
Secrets can be stored in a separate secret manager and referenced in the Terraform configuration. 
41. What is a Terraform manifest? 
A Terraform manifest is a configuration file that contains the code for provisioning resources. 
42. How do you use the override file? 
An override file lets you override resource or provider configurations without modifying the original source. 
43. How do you manage resource immutability? 
You manage resource immutability by using versioning and a destroy_then_create lifecycle. 
44. What is a Terraform variable? 
A variable is a value that can be defined at runtime to customize a configuration. 45. How do you manage versions in your code? 
You manage versions using a version control system like Git and tagging releases. 46. How do you handle state file corruption? 
You handle state file corruption by restoring from a backup, manually editing the state
file, or using the terraform state command. 
47. What is the required_version? 
required_version specifies the minimum compatible version of Terraform to run the configuration. 
48. What are remote-exec and local-exec? 
remote-exec runs a command on a remote resource, while local-exec runs a command on the machine executing Terraform. 
49. What is the output value? 
An output value exposes data from the Terraform configuration, such as a public IP address or a DNS name. 
50. How do you manage multiple clouds with Terraform? 
By using multiple provider blocks, modules, and a single repository. 
51. How do you debug Terraform configurations? 
You debug configurations by enabling detailed logs, using terraform console, and checking the state file. 
52. How do you manage infrastructure as a product? 
By defining clear modules, versioning, and a well-documented process. 53. How do you test Terraform configurations? 
You can test configurations using unit tests, integration tests, and static analysis tools. 54. How do you handle multiple state files? 
You handle multiple state files using workspaces, separate folders, or a remote backend. 55. How do you perform a dry run in Terraform? 
You perform a dry run using terraform plan, which shows the changes without applying them. 
56. How do you manage versioning of Terraform modules? 
By tagging module repositories with versions and referencing them in the source block. 57. How do you use count vs for_each? 
You use count for simple numeric loops, while for_each is used for more complex scenarios with maps and sets. 
58. What is a terraform plan file? 
A plan file is a binary file that stores the result of terraform plan for later use in terraform apply. 
59. How do you manage secrets in Terraform Cloud? 
You manage secrets in Terraform Cloud by using variables marked as sensitive. 60. What is a dynamic block? 
A dynamic block generates nested configuration blocks based on a complex data type. 61. What are the benefits of using a remote state? 
Benefits include collaboration, state locking, security, and a single source of truth for all teams. 
62. What is the purpose of terraform taint? 
terraform taint marks a resource for destruction and recreation on the next apply. 63. What is the use of terraform graph? 
terraform graph generates a visual graph of the resources and their dependencies.
64. How do you perform a rollback in Terraform? 
You perform a rollback by reverting to a previous version of the code and running terraform apply. 
65. What is the null_resource? 
A null_resource is a resource that does not map to any real-world infrastructure. 66. What is a terraform plan -destroy? 
terraform plan -destroy shows the resources that will be deleted without actually destroying them. 
67. What are the common Terraform errors? 
Common errors include syntax errors, provider authentication issues, and state file corruption. 
68. What are the advantages of using Terraform over other tools? 
Advantages include multi-cloud support, a declarative language, and a large community. 69. What is terraform console used for? 
terraform console is used for testing expressions and debugging your code. 70. How do you manage Terraform modules in a private repository? 
You manage modules by referencing a Git repository with an SSH key or a private registry. 
71. What is state locking? 
State locking prevents multiple users from modifying the state file at the same time, avoiding conflicts. 
72. What are sentinel policies? 
Sentinel policies are a framework for enforcing governance and policy as code. 73. How do you manage a large-scale Terraform codebase? 
By using modules, workspaces, a remote backend, and a CI/CD pipeline. 74. What is terraform import? 
terraform import adds existing infrastructure to a Terraform state file. 75. What are the risks of using Terraform? 
Risks include state file corruption, accidental deletion of resources, and provider API limits. 
76. What are the best practices for using Terraform? 
Use a remote backend, modules, version control, and a CI/CD pipeline. 77. How do you manage state file backups? 
By using a remote backend that supports state versioning and regular backups. 78. What is the difference between count and for_each? 
count is for simple numeric loops, while for_each is used for complex scenarios with maps and sets. 
79. What is a resource in Terraform? 
A resource is a block that defines a physical resource to be created in the cloud. 80. What is the terraform state command? 
terraform state is a command for managing and inspecting the state file. 81. What is the provider block? 
The provider block configures the cloud provider, such as AWS or Azure.
82. How do you use the output value? 
output values are used to expose data from the configuration, such as a public IP address. 
83. What is the terraform graph? 
terraform graph generates a visual graph of resources and their dependencies. 84. How do you perform a dry run in Terraform? 
You perform a dry run using terraform plan, which shows the changes without applying them. 
85. How do you manage resource dependencies? 
By using implicit dependencies, which Terraform infers, or explicit dependencies with depends_on. 
86. What is terraform validate? 
terraform validate checks for syntax errors and consistency in the configuration. 87. What is the lifecycle block used for? 
The lifecycle block is used to control resource creation, update, and deletion behavior. 88. What are the benefits of using modules? 
Benefits include reusability, consistency, and better organization of code. 89. How do you manage variables in Terraform? 
By defining them in a variables.tf file and passing values via a tfvars file or command-line flags. 
90. What is a Terraform module registry? 
A module registry is a repository for sharing and discovering Terraform modules. 91. What is terraform state rm? 
terraform state rm removes a resource from the state file without destroying the actual resource. 
92. What are the challenges in cross-cloud deployments? 
Different provider capabilities, complex networking, and maintaining consistency require careful module design. 
93. What are the most common debugging scenarios you have faced? Circular dependencies and drift that required splitting the infrastructure into modules and manually editing the state. 
94. How do you estimate costs in Terraform? 
You use tools like InfraCost that pass Terraform plan outputs to estimate resource costs. 95. How do you prevent accidental deletions? 
You use lifecycle prevent_destroy, code reviews, approvals, and regular backups. 96. How do you perform performance tuning? 
You adjust parallelism, configure providers efficiently, avoid unnecessary resources, and reuse modules. 
97. How do you add custom validation rules? 
You use variable blocks with validation expressions to catch errors early. 98. How do you troubleshoot scalability issues? 
You split state, adjust parallelism, and modularize infrastructure to handle scale issues. 99. How do you manage and update providers?
You manage and update providers using the required_providers block and terraform init -upgrade. 
100. How do you manage dynamic secrets in Terraform? 
By using a secret manager that can provide secrets at runtime to your Terraform configuration.


When you created the environment using Terraform, what components did you set up using Terraform?

How do you make changes to the configuration of already created resources using Terraform?
-> we can use terraform import to do that
 When the Terraform state file is created, what do you do with that state file and where do you store and find it?
-> terror maintains a state file that maps the current status of your infra with with config file. You can store it in local m/c, we can store it in s3 or Terraform cloud. By default it is stored in local m/c and named as tarrraform.tfstate. It has sensitive info so it is advised to store it in encrypted env.

How do you resolve the issue if you lose the Terraform state file?


What are the major features you have found in Terraform that you can talk about?
What are the major features you have found in Terraform that you can talk about?
What is the `terraform validate` command used for, and can you provide an example?
Have you ever heard about the lifecycle in Terraform? Can you talk more about it?
Have you worked with tools like CloudFormation, Ansible, or anything similar?
Do you have any experience with Ansible?
If you had to choose between Ansible and Terraform, which one would you prefer and why?
In your current organization, which tool are you using: Ansible, Terraform, or Pulumi?
Can you talk about any features of Pulumi that you find particularly useful or impressive?
Have you ever heard about Bicep or ARM templates?
In a scenario where you have 20 resources running on a public cloud (AWS or Azure) and you want to destroy just one resource, how would you go about doing that?
Have you ever preserved a key that you created using Terraform?
What happens if you delete the Terraform state file and then run the `terraform apply` or `terraform plan` command?
Have you ever worked with modules in Terraform?
What are the different types of modules in Terraform?
The module that gets called is what: a parent module or a child module?
From where we call a module, what is that module called?
Have you ever heard about remote backends in Terraform? Do you have any ideas or experience with them?
How can you provide variable values at runtime in Terraform?
In an organization, how do you manage multiple environments in Terraform?
Why do we call Terraform "Infrastructure as Code" (IaC)? Is there a particular reason for this?
Can you explain some drawbacks or challenges you have faced in your career?
Which version of Terraform are you using?



I want to remove duplicates from below quesions and want only unique question swith its answers ina file-

I have duplicate questions below text, will you help me keep only 1 copy of a question and delete the duplicates?
 
1. What is Terraform?
Terraform is an open-source infrastructure as code (IaC) tool developed by HashiCorp that allows
users to define and provision data center infrastructure using a high-level configuration language
called HCL (HashiCorp Configuration Language) or JSON.
Example:
provider "aws" {
region = "us-west-2"
}
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
2. What are the main features of Terraform?
Terraform's main features include Infrastructure as Code (IaC), execution plans, resource graphs,
change automation, and state management.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
3. What is the difference between Terraform and other IaC tools like
Ansible, Puppet, and Chef?
Terraform focuses on infrastructure provisioning, is declarative, and uses HCL. Tools like Ansible,
Puppet, and Chef focus on configuration management and are procedural.
Example:
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
4. What is a provider in Terraform?
A provider is a plugin that Terraform uses to manage an external API. Providers define the
resources and data sources available.
Example:
provider "aws" {
region = "us-west-2"
}
5. How does Terraform manage dependencies?
Terraform uses a dependency graph to manage dependencies between resources. It
automatically understands the order of operations needed based on resource dependencies.
Example:
resource "aws_instance" "web" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
subnet_id = aws_subnet.example.id
}
resource "aws_subnet" "example" {
vpc_id = aws_vpc.example.id
cidr_block = "10.0.1.0/24"
}
resource "aws_vpc" "example" {
cidr_block = "10.0.0.0/16"
}
6. What is a state file in Terraform?
A state file is a file that Terraform uses to keep track of the current state of the infrastructure. It
maps the resources defined in the configuration to the real-world resources.
Example:
terraform show
7. Why is it important to manage the state file in Terraform?
Managing the state file is crucial because it ensures consistency between the infrastructure's real
state and the configuration. It also enables features like change detection and planning.
Example:
terraform init
8. How can you secure the state file in Terraform?
State files can be secured by storing them in remote backends with proper access controls and
encryption, such as AWS S3 with server-side encryption and access control policies.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
encrypt = true
}
}
9. What are modules in Terraform?
Modules are reusable packages of Terraform configurations that can be shared and composed to
manage resources efficiently.
Example:
module "vpc" {
source = "./modules/vpc"
}
10. What is the purpose of the terraform init command?
terraform init initializes a working directory containing Terraform configuration files,
downloads the necessary provider plugins, and prepares the environment.
Example:
terraform init
11. What does the terraform plan command do?
terraform plan creates an execution plan, showing what actions Terraform will take to achieve
the desired state defined in the configuration.
Example:
terraform plan
12. What is the terraform apply command used for?
terraform apply applies the changes required to reach the desired state of the configuration.
It executes the plan created by terraform plan.
Example:
terraform apply
13. What is the purpose of the terraform destroy command?
terraform destroy is used to destroy the infrastructure managed by Terraform. It removes all
the resources defined in the configuration.
Example:
terraform destroy
14. How do you define and use variables in Terraform?
Variables in Terraform are defined using the variable block and can be used by referring to
them with var.<variable_name>.
Example:
variable "instance_type" {
description = "Type of EC2 instance"
default = "t2.micro"
}
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = var.instance_type
}
15. What are output values in Terraform and how are they used?
Output values are used to extract information from the resources and make it accessible after
the apply phase. They can be used to output resource attributes.
Example:
output "instance_id" {
value = aws_instance.example.id
}
16. How do you manage different environments (e.g., dev, prod) in
Terraform?
Different environments can be managed using workspaces or separate directories with different
variable files and state files.
Example:
terraform workspace new dev
terraform workspace new prod
17. What is remote state and how do you configure it in Terraform?
Remote state allows Terraform to store the state file in a remote storage backend, enabling team
collaboration and secure storage.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
18. How do you import existing resources into Terraform?
Existing resources can be imported using the terraform import command, which maps the
existing resource to a Terraform resource in the state file.
Example:
terraform import aws_instance.example i-1234567890abcdef0
19. What are data sources in Terraform?
Data sources allow Terraform to fetch data from existing infrastructure or services to use in
resource definitions.
Example:
data "aws_ami" "example" {
most_recent = true
owners = ["amazon"]
filter {
name = "name"
values = ["amzn-ami-hvm-*"]
}
}
20. What are provisioners in Terraform?
Provisioners are used to execute scripts or commands on a local or remote machine as part of
the resource lifecycle.
Example:
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
provisioner "local-exec" {
command = "echo ${self.public_ip} > ip_address.txt"
}
}
21. How do you handle secrets in Terraform?
Secrets can be managed using environment variables, secure secret management services (e.g.,
AWS Secrets Manager), or Terraform's sensitive attribute.
Example:
resource "aws_secretsmanager_secret" "example" {
name = "example"
description = "An example secret"
}
resource "aws_secretsmanager_secret_version" "example" {
secret_id = aws_secretsmanager_secret.example.id
secret_string = jsonencode({
username = "example_user"
password = "example_password"
})
}
22. What is a backend in Terraform?
A backend in Terraform defines where and how state is loaded and stored. It can be local or
remote (e.g., S3, Consul, etc.).
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
23. How do you use conditional expressions in Terraform?
Conditional expressions in Terraform are used to assign values based on conditions using the
ternary operator condition ? true_value : false_value.
Example:
variable "environment" {
default = "dev"
}
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"
}
24. What is the purpose of the terraform validate command?
terraform validate is used to validate the syntax and configuration of the Terraform files
without creating any resources.
Example:
terraform validate
25. How can you format Terraform configuration files?
Terraform configuration files can be formatted using the terraform fmt command, which
formats the files according to the
Terraform style guide.
Example:
terraform fmt
26. What is the difference between count and for_each in Terraform?
count is used to create multiple instances of a resource, while for_each is used to iterate over a
map or set of values to create multiple instances.
Example (count):
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
Example (for_each):
resource "aws_instance" "example" {
for_each = toset(["instance1", "instance2"])
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
27. How do you use loops in Terraform?
Loops in Terraform can be implemented using the count and for_each meta-arguments, as
well as the for expression in variable assignments.
Example:
variable "instance_names" {
type = list(string)
default = ["instance1", "instance2"]
}
resource "aws_instance" "example" {
for_each = toset(var.instance_names)
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
28. What are locals in Terraform and how do you use them?
Locals in Terraform are used to define local values that can be reused within a module. They help
avoid repetition and make configurations more readable.
Example:
locals {
instance_type = "t2.micro"
ami_id = "ami-0c55b159cbfafe1f0"
}
resource "aws_instance" "example" {
ami = local.ami_id
instance_type = local.instance_type
}
29. What is the purpose of the terraform taint command?
terraform taint marks a resource for recreation on the next terraform apply. It is useful
when a resource needs to be replaced due to a manual change or corruption.
Example:
terraform taint aws_instance.example
30. How do you manage module versioning in Terraform?
Module versioning in Terraform can be managed using the version argument in
the source attribute of a module block, typically in combination with a registry.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
31. What is the Terraform Registry?
The Terraform Registry is a public repository of Terraform modules and providers that can be
used to discover and use pre-built modules and providers.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
32. How do you perform a dry run in Terraform?
A dry run in Terraform can be performed using the terraform plan command, which shows
the execution plan without making any changes.
Example:
terraform plan
33. What is the terraform state command used for?
The terraform state command is used to manage and manipulate the state file. It provides
subcommands to move, remove, list, and inspect resources in the state file.
Example:
terraform state list
34. How do you rename a resource in the state file?
A resource can be renamed in the state file using the terraform state mv command, which
moves the state of a resource to a new address.
Example:
terraform state mv aws_instance.old_name aws_instance.new_name
35. What is the purpose of the terraform workspace command?
The terraform workspace command is used to manage multiple workspaces, allowing for
different states to be associated with the same configuration.
Example:
terraform workspace new dev
terraform workspace select dev
36. How do you debug Terraform configurations?
Debugging Terraform configurations can be done using the TF_LOG environment variable to set
the log level and the terraform console command to interact with the configuration.
Example:
export TF_LOG=DEBUG
terraform apply
37. What is the difference between local and remote backends in
Terraform?
Local backends store the state file on the local filesystem, while remote backends store the state
file in a remote storage service (e.g., S3, Consul).
Example (local backend):
terraform {
backend "local" {
path = "terraform.tfstate"
}
}
Example (remote backend):
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
38. How do you handle provider versioning in Terraform?
Provider versioning in Terraform is managed using the required_providers block in
the terraform block, specifying the version constraints.
Example:
terraform {
required_providers {
aws = {
source = "hashicorp/aws"
version = "~> 3.0"
}
}
}
39. What is the purpose of the terraform refresh command?
terraform refresh updates the state file with the current state of the infrastructure without
making any changes to the configuration.
Example:
terraform refresh
40. How do you generate and view a resource graph in Terraform?
A resource graph can be generated using the terraform graph command and can be viewed
using tools like Graphviz.
Example:
terraform graph | dot -Tpng > graph.png
41. What are lifecycle blocks in Terraform?
lifecycle blocks in Terraform are used to customize the lifecycle of a resource, such as creating
before destroying, ignoring changes, and preventing deletion.
Example:
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
lifecycle {
create_before_destroy = true
}
}
42. How do you ignore changes to a resource attribute in Terraform?
Changes to a resource attribute can be ignored using the ignore_changes argument in
a lifecycle block.
Example:
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
lifecycle {
ignore_changes = [ami]
}
}
43. What is the terraform import command used for?
terraform import is used to import existing infrastructure into Terraform's state file, mapping
it to resources defined in the configuration.
Example:
terraform import aws_instance.example i-1234567890abcdef0
44. How do you use output values across different modules in Terraform?
Output values from one module can be referenced in another module by using the module's
output attributes.
Example:
module "vpc" {
source = "./modules/vpc"
}
output "vpc_id" {
value = module.vpc.vpc_id
}
45. What is the difference between terraform output and output values in
configuration?
terraform output is a command that displays the output values of a Terraform configuration,
while output values in configuration are defined using the output block.
Example (command):
terraform output
Example (configuration):
output "instance_id" {
value = aws_instance.example.id
}
46. What are dynamic blocks in Terraform?
Dynamic blocks in Terraform are used to generate multiple nested blocks within a resource or
module based on dynamic content.
Example:
resource "aws_security_group" "example" {
name = "example-sg"
description = "Example security group"
dynamic "ingress" {
for_each = var.ingress_rules
content {
from_port = ingress.value.from_port
to_port = ingress.value.to_port
protocol = ingress.value.protocol
cidr_blocks = ingress.value.cidr_blocks
}
}
}
47. How do you define and use maps in Terraform?
Maps in Terraform are defined using the map type and can be used to store key-value pairs. They
are accessed using the key.
Example:
variable "ami_ids" {
type = map(string)
default = {
us-east-1 = "ami-0c55b159cbfafe1f0"
us-west-2 = "ami-0d5eff06f840b45e9"
}
}
resource "aws_instance" "example" {
ami = var.ami_ids[var.region]
instance_type = "t2.micro"
}
48. What is a count parameter in Terraform?
The
count parameter in Terraform is used to create multiple instances of a resource based on a
specified number.
Example:
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
49. What are Terraform Cloud and Terraform Enterprise?
Terraform Cloud and Terraform Enterprise are commercial versions of Terraform that provide
collaboration, governance, and automation features.
Example:
terraform {
backend "remote" {
organization = "my-org"
workspaces {
name = "my-workspace"
}
}
}
50. How do you use a Terraform backend?
A backend is configured using the terraform block in the configuration file, specifying the
backend type and its configuration.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
51. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
52. How do you use a lock file in Terraform?
A lock file (.terraform.lock.hcl) is used to lock provider versions, ensuring consistency in
provider versions across different environments.
Example:
terraform init
53. What is the purpose of the terraform workspace command?
The terraform workspace command is used to create, select, and manage multiple
workspaces, allowing different states to be associated with the same configuration.
Example:
terraform workspace new dev
terraform workspace select dev
54. How do you manage secrets in Terraform?
Secrets can be managed using environment variables, secure secret management services (e.g.,
AWS Secrets Manager), or Terraform's sensitive attribute.
Example:
resource "aws_secretsmanager_secret" "example" {
name = "example"
description = "An example secret"
}
resource "aws_secretsmanager_secret_version" "example" {
secret_id = aws_secretsmanager_secret.example.id
secret_string = jsonencode({
username = "example_user"
password = "example_password"
})
}
55. What is the terraform console command used for?
terraform console opens an interactive console for evaluating expressions, testing
interpolation syntax, and debugging configurations.
Example:
terraform console
56. How do you reference data sources in Terraform?
Data sources are referenced using the data block and can be used to fetch information about
existing infrastructure or services.
Example:
data "aws_ami" "example" {
most_recent = true
owners = ["amazon"]
filter {
name = "name"
values = ["amzn-ami-hvm-*"]
}
}
resource "aws_instance" "example" {
ami = data.aws_ami.example.id
instance_type = "t2.micro"
}
57. What is the purpose of the terraform state mv command?
terraform state mv moves a resource in the state file to a new address, useful for renaming
resources without recreating them.
Example:
terraform state mv aws_instance.old_name aws_instance.new_name
58. How do you use conditional expressions in Terraform?
Conditional expressions in Terraform are used to assign values based on conditions using the
ternary operator condition ? true_value : false_value.
Example:
variable "environment" {
default = "dev"
}
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"
}
59. What is the terraform taint command used for?
terraform taint marks a resource for recreation on the next terraform apply. It is useful
when a resource needs to be replaced due to a manual change or corruption.
Example:
terraform taint aws_instance.example
60. How do you define and use maps in Terraform?
Maps in Terraform are defined using the map type and can be used to store key-value pairs. They
are accessed using the key.
Example:
variable "ami_ids" {
type = map(string)
default = {
us-east-1 = "ami-0c55b159cbfafe1f0"
us-west-2 = "ami-0d5eff06f840b45e9"
}
}
resource "aws_instance" "example" {
ami = var.ami_ids[var.region]
instance_type = "t2.micro"
}
61. How do you handle provider versioning in Terraform?
Provider versioning in Terraform is managed using the required_providers block in
the terraform block, specifying the version constraints.
Example:
terraform {
required_providers {
aws = {
source = "hashicorp/aws"
version = "~> 3.0"
}
}
}
62. What is the purpose of the terraform refresh command?
terraform refresh updates the state file with the current state of the infrastructure without
making any changes to the configuration.
Example:
terraform refresh
63. What are lifecycle blocks in Terraform?
lifecycle blocks in Terraform are used to customize the lifecycle of a resource, such as creating
before destroying, ignoring changes, and preventing deletion.
Example:
resource "aws_instance" "example" {
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
lifecycle {
create_before_destroy = true
}
}
64. How do you use loops in Terraform?
Loops in Terraform can be implemented using the count and for_each meta-arguments, as
well as the for expression in variable assignments.
Example:
variable "instance_names" {
type = list(string)
default = ["instance1", "instance2"]
}
resource "aws_instance" "example" {
for_each = toset(var.instance_names)
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
65. What is the terraform import command used for?
terraform import is used to import existing infrastructure into Terraform's state file, mapping
it to resources defined in the configuration.
Example:
terraform import aws_instance.example i-1234567890abcdef0
66. How do you use output values across different modules in Terraform?
Output values from one module can be referenced in another module by using the module's
output attributes.
Example:
module "vpc" {
source = "./modules/vpc"
}
output "vpc_id" {
value = module.vpc.vpc_id
}
67. What is the difference between terraform output and output values in
configuration?
terraform output is a command that displays the output values of a Terraform configuration,
while output values in configuration are defined using the output block.
Example (command):
terraform output
Example (configuration):
output "instance_id" {
value = aws_instance.example.id
}
68. What are dynamic blocks in Terraform?
Dynamic blocks in Terraform are used to generate multiple nested blocks within a resource or
module based on dynamic content.
Example:
resource "aws_security_group" "example" {
name = "example-sg"
description = "Example security group"
dynamic "ingress" {
for_each = var.ingress_rules
content {
from_port = ingress.value.from_port
to_port = ingress.value.to_port
protocol = ingress.value.protocol
cidr_blocks = ingress.value.cidr_blocks
}
}
}
69. How do you manage different environments (e.g., dev, prod) in
Terraform?
Different environments can be managed using workspaces or separate directories with different
variable files and state files.
Example:
terraform workspace new dev
terraform workspace new prod
70. How do you handle secrets in Terraform?
Secrets can be managed using environment variables, secure secret management services (e.g.,
AWS Secrets Manager), or Terraform's sensitive attribute.
Example:
resource "aws_secretsmanager_secret" "example" {
name = "example"
description = "An example secret"
}
resource "aws_secretsmanager_secret_version" "example" {
secret_id = aws_secretsmanager_secret.example.id
secret_string = jsonencode({
username = "example_user"
password = "example_password"
})
}
71. What is the terraform console command used for?
terraform console opens an interactive console for evaluating expressions, testing
interpolation syntax, and debugging
configurations.
Example:
terraform console
72. How do you reference data sources in Terraform?
Data sources are referenced using the data block and can be used to fetch information about
existing infrastructure or services.
Example:
data "aws_ami" "example" {
most_recent = true
owners = ["amazon"]
filter {
name = "name"
values = ["amzn-ami-hvm-*"]
}
}
resource "aws_instance" "example" {
ami = data.aws_ami.example.id
instance_type = "t2.micro"
}
73. What is the purpose of the terraform state mv command?
terraform state mv moves a resource in the state file to a new address, useful for renaming
resources without recreating them.
Example:
terraform state mv aws_instance.old_name aws_instance.new_name
74. What is a backend in Terraform?
A backend in Terraform defines where and how state is loaded and stored. It can be local or
remote (e.g., S3, Consul, etc.).
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
 
76. What is the difference between local and remote backends in
Terraform?
Local backends store the state file on the local filesystem, while remote backends store the state
file in a remote storage service (e.g., S3, Consul).
Example (local backend):
terraform {
backend "local" {
path = "terraform.tfstate"
}
}
Example (remote backend):
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
77. How do you manage module versioning in Terraform?
Module versioning in Terraform can be managed using the version argument in
the source attribute of a module block, typically in combination with a registry.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
78. What is the Terraform Registry?
The Terraform Registry is a public repository of Terraform modules and providers that can be
used to discover and use pre-built modules and providers.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
79. How do you generate and view a resource graph in Terraform?
A resource graph can be generated using the terraform graph command and can be viewed
using tools like Graphviz.
Example:
terraform graph | dot -Tpng > graph.png
80. What is the purpose of the terraform validate command?
terraform validate is used to validate the syntax and configuration of the Terraform files
without creating any resources.
Example:
terraform validate
81. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
82. What are locals in Terraform and how do you use them?
Locals in Terraform are used to define local values that can be reused within a module. They help
avoid repetition and make configurations more readable.
Example:
locals {
instance_type = "t2.micro"
ami_id = "ami-0c55b159cbfafe1f0"
}
resource "aws_instance" "example" {
ami = local.ami_id
instance_type = local.instance_type
}
83. How do you handle provider dependencies in Terraform?
Provider dependencies in Terraform are managed using the required_providers block in
the terraform block, specifying the version constraints.
Example:
terraform {
required_providers {
aws = {
source = "hashicorp/aws"
version = "~> 3.0"
}
}
}
84. What is the terraform apply command used for?
terraform apply applies the changes required to reach the desired state of the configuration.
It executes the plan created by terraform plan.
Example:
terraform apply
85. What is the terraform destroy command used for?
terraform destroy is used to destroy the infrastructure managed by Terraform. It removes all
the resources defined in the configuration.
Example:
terraform destroy
86. What are output values in Terraform and how are they used?
Output values are used to extract information from the resources and make it accessible after
the apply phase. They can be used to output resource attributes.
Example:
output "instance_id" {
value = aws_instance.example.id
}
87. How do you manage different environments (e.g., dev, prod) in
Terraform?
Different environments can be managed using workspaces or separate directories with different
variable files and state files.
Example:
terraform workspace new dev
terraform workspace new prod
88. What is the difference between count and for_each in Terraform?
count is used to create multiple instances of a resource, while for_each is used to iterate over a
map or set of values to create multiple instances.
Example (count):
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
Example (for_each):
resource "aws_instance" "example" {
for_each = toset(["instance1", "instance2"])
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
89. How do you use loops in Terraform?
Loops in Terraform can be implemented using the count and for_each meta-arguments, as
well as the for expression in variable assignments.
Example:
variable "instance_names" {
type = list(string)
default = ["instance1", "instance2"]
}
resource "aws_instance" "example" {
for_each = toset(var.instance_names)
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
90. What is a count parameter in Terraform?
The count parameter in Terraform is used to create multiple instances of a resource based on a
specified number.
Example:
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
91. What are Terraform Cloud and Terraform Enterprise?
Terraform Cloud and Terraform Enterprise are commercial versions of Terraform that provide
collaboration, governance, and automation features.
Example:
terraform {
backend "remote" {
organization = "my-org"
workspaces {
name = "my-workspace"
}
}
}
92. How do you use a Terraform backend?
A backend is configured using the terraform block in the configuration file, specifying the
backend type and its configuration.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
93. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
 
95. What is the purpose of the terraform workspace command?
The terraform workspace command is used to create, select, and manage multiple
workspaces, allowing different states to be associated with the same configuration.
Example:
terraform workspace new dev
terraform workspace select dev
96. **How do you manage secrets
in Terraform?**
Secrets can be managed using environment variables, secure secret management services (e.g.,
AWS Secrets Manager), or Terraform's sensitive attribute.
Example:
resource "aws_secretsmanager_secret" "example" {
name = "example"
description = "An example secret"
}
resource "aws_secretsmanager_secret_version" "example" {
secret_id = aws_secretsmanager_secret.example.id
secret_string = jsonencode({
username = "example_user"
password = "example_password"
})
}
97. What is the terraform console command used for?
terraform console opens an interactive console for evaluating expressions, testing
interpolation syntax, and debugging configurations.
Example:
terraform console
98. How do you reference data sources in Terraform?
Data sources are referenced using the data block and can be used to fetch information about
existing infrastructure or services.
Example:
data "aws_ami" "example" {
most_recent = true
owners = ["amazon"]
filter {
name = "name"
values = ["amzn-ami-hvm-*"]
}
}
resource "aws_instance" "example" {
ami = data.aws_ami.example.id
instance_type = "t2.micro"
}
99. What is the purpose of the terraform state mv command?
terraform state mv moves a resource in the state file to a new address, useful for renaming
resources without recreating them.
Example:
terraform state mv aws_instance.old_name aws_instance.new_name
100. What is a backend in Terraform?
A backend in Terraform defines where and how state is loaded and stored. It can be local or
remote (e.g., S3, Consul, etc.).
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
102. What is the difference between local and remote backends in
Terraform?
Local backends store the state file on the local filesystem, while remote backends store the state
file in a remote storage service (e.g., S3, Consul).
Example (local backend):
terraform {
backend "local" {
path = "terraform.tfstate"
}
}
Example (remote backend):
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
103. How do you manage module versioning in Terraform?
Module versioning in Terraform can be managed using the version argument in
the source attribute of a module block, typically in combination with a registry.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
104. What is the Terraform Registry?
The Terraform Registry is a public repository of Terraform modules and providers that can be
used to discover and use pre-built modules and providers.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
105. How do you generate and view a resource graph in Terraform?
A resource graph can be generated using the terraform graph command and can be viewed
using tools like Graphviz.
Example:
terraform graph | dot -Tpng > graph.png
106. What is the purpose of the terraform validate command?
terraform validate is used to validate the syntax and configuration of the Terraform files
without creating any resources.
Example:
terraform validate
107. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
108. What are locals in Terraform and how do you use them?
Locals in Terraform are used to define local values that can be reused within a module. They help
avoid repetition and make configurations more readable.
Example:
locals {
instance_type = "t2.micro"
ami_id = "ami-0c55b159cbfafe1f0"
}
resource "aws_instance" "example" {
ami = local.ami_id
instance_type = local.instance_type
}
109. How do you handle provider dependencies in Terraform?
Provider dependencies in Terraform are managed using the required_providers block in
the terraform block, specifying the version constraints.
Example:
terraform {
required_providers {
aws = {
source = "hashicorp/aws"
version = "~> 3.0"
}
}
}
110. What is the terraform apply command used for?
terraform apply applies the changes required to reach the desired state of the configuration.
It executes the plan created by terraform plan.
Example:
terraform apply
111. What is the terraform destroy command used for?
terraform destroy is used to destroy the infrastructure managed by Terraform. It removes all
the resources defined in the configuration.
Example:
terraform destroy
112. What are output values in Terraform and how are they used?
Output values are used to extract information from the resources and make it accessible after
the apply phase. They can be used to output resource attributes.
Example:
output "instance_id" {
value = aws_instance.example.id
}
113. How do you manage different environments (e.g., dev, prod) in
Terraform?
Different environments can be managed using workspaces or separate directories with different
variable files and state files.
Example:
terraform workspace new dev
terraform workspace new prod
114. What is the difference between count and for_each in Terraform?
count is used to create multiple instances of a resource, while for_each is used to iterate over a
map or set of values to create multiple instances.
Example (count):
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
Example (for_each):
resource "aws_instance" "example" {
for_each = toset(["instance1", "instance2"])
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
115. How do you use loops in Terraform?
Loops in Terraform can be implemented using the count and for_each meta-arguments, as
well as the for expression in variable assignments.
Example:
variable "instance_names" {
type = list(string)
default = ["instance1", "instance2"]
}
resource "aws_instance" "example" {
for_each = toset(var.instance_names)
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
116. What is a count parameter in Terraform?
The count parameter in Terraform is used to create multiple instances of a resource based on a
specified number.
Example:
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
117. What are Terraform Cloud and Terraform Enterprise?
Terraform Cloud and Terraform Enterprise are commercial versions of Terraform that provide
collaboration, governance, and automation features.
Example:
terraform {
backend "remote" {
organization = "my-org"
workspaces {
name = "my-workspace"
}
}
}
118. How do you use a Terraform backend?
A backend is configured using the terraform block in the configuration file, specifying the
backend type and its configuration.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
119. What is the terraform fmt command used for?
terraform fmt formats the configuration files
to follow the Terraform style guide, making the code consistent and readable.
Example:
terraform fmt
 
121. What is the purpose of the terraform workspace command?
The terraform workspace command is used to create, select, and manage multiple
workspaces, allowing different states to be associated with the same configuration.
Example:
terraform workspace new dev
terraform workspace select dev
122. How do you manage secrets in Terraform?
 
123. What is the terraform console command used for?
terraform console opens an interactive console for evaluating expressions, testing
interpolation syntax, and debugging configurations.
Example:
terraform console
124. How do you reference data sources in Terraform?
Data sources are referenced using the data block and can be used to fetch information about
existing infrastructure or services.
Example:
data "aws_ami" "example" {
most_recent = true
owners = ["amazon"]
filter {
name = "name"
values = ["amzn-ami-hvm-*"]
}
}
resource "aws_instance" "example" {
ami = data.aws_ami.example.id
instance_type = "t2.micro"
}
125. What is the purpose of the terraform state mv command?
terraform state mv moves a resource in the state file to a new address, useful for renaming
resources without recreating them.
Example:
terraform state mv aws_instance.old_name aws_instance.new_name
126. What is a backend in Terraform?
A backend in Terraform defines where and how state is loaded and stored. It can be local or
remote (e.g., S3, Consul, etc.).
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
 
128. What is the difference between local and remote backends in
Terraform?
Local backends store the state file on the local filesystem, while remote backends store the state
file in a remote storage service (e.g., S3, Consul).
Example (local backend):
terraform {
backend "local" {
path = "terraform.tfstate"
}
}
Example (remote backend):
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
129. How do you manage module versioning in Terraform?
Module versioning in Terraform can be managed using the version argument in
the source attribute of a module block, typically in combination with a registry.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
130. What is the Terraform Registry?
The Terraform Registry is a public repository of Terraform modules and providers that can be
used to discover and use pre-built modules and providers.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
131. How do you generate and view a resource graph in Terraform?
A resource graph can be generated using the terraform graph command and can be viewed
using tools like Graphviz.
Example:
terraform graph | dot -Tpng > graph.png
132. What is the purpose of the terraform validate command?
terraform validate is used to validate the syntax and configuration of the Terraform files
without creating any resources.
Example:
terraform validate
133. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
134. What are locals in Terraform and how do you use them?
Locals in Terraform are used to define local values that can be reused within a module. They help
avoid repetition and make configurations more readable.
Example:
locals {
instance_type = "t2.micro"
ami_id = "ami-0c55b159cbfafe1f0"
}
resource "aws_instance" "example" {
ami = local.ami_id
instance_type = local.instance_type
}
135. How do you handle provider dependencies in Terraform?
Provider dependencies in Terraform are managed using the required_providers block in
the terraform block, specifying the version constraints.
Example:
terraform {
required_providers {
aws = {
source = "hashicorp/aws"
version = "~> 3.0"
}
}
}
136. What is the terraform apply command used for?
terraform apply applies the changes required to reach the desired state of the configuration.
It executes the plan created by terraform plan.
Example:
terraform apply
137. What is the terraform destroy command used for?
terraform destroy is used to destroy the infrastructure managed by Terraform. It removes all
the resources defined in the configuration.
Example:
terraform destroy
138. What are output values in Terraform and how are they used?
Output values are used to extract information from the resources and make it accessible after
the apply phase. They can be used to output resource attributes.
Example:
output "instance_id" {
value = aws_instance.example.id
}
139. How do you manage different environments (e.g., dev, prod) in
Terraform?
Different environments can be managed using workspaces or separate directories with different
variable files and state files.
Example:
terraform workspace new dev
terraform workspace new prod
140. What is the difference between count and for_each in Terraform?
count is used to create multiple instances of a resource, while for_each is used to iterate over a
map or set of values to create multiple instances.
Example (count):
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
Example (for_each):
resource "aws_instance" "example" {
for_each = toset(["instance1", "instance2"])
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
143. What are Terraform Cloud and Terraform Enterprise?
Terraform Cloud and Terraform Enterprise are commercial versions of Terraform that provide
collaboration, governance, and automation features.
Example:
terraform {
backend "remote" {
organization = "my-org"
workspaces {
name = "my-workspace"
}
}
}
144. How do you use a Terraform backend?
A backend is configured using the terraform block in the configuration file, specifying the
backend type and its configuration.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
145. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
 
147. What is the purpose of the terraform workspace command?
The terraform workspace command is used to create, select, and manage multiple
workspaces, allowing different states to be associated with the same configuration.
Example:
terraform workspace new dev
terraform workspace select dev
148. How do you manage secrets in Terraform?
Secrets can be managed using environment variables, secure secret management services (e.g.,
AWS Secrets Manager), or Terraform's sensitive attribute.
Example:
resource "aws_secretsmanager_secret" "example" {
name = "example"
description = "An example secret"
}
resource "aws_secretsmanager_secret_version" "example" {
secret_id = aws_secretsmanager_secret.example.id
secret_string = jsonencode({
username = "example_user"
password = "example_password"
})
}
149. What is the terraform console command used for?
terraform console opens an interactive console for evaluating expressions, testing
interpolation syntax, and debugging configurations.
Example:
terraform console
150. How do you reference data sources in Terraform?
Data sources are referenced using the data block and can be used to fetch information about
existing infrastructure or services.
Example:
data "aws_ami" "example" {
most_recent = true
owners = ["amazon"]
filter {
name = "name"
values = ["amzn-ami-hvm-*"]
}
}
resource "aws_instance" "example" {
ami = data.aws_ami.example.id
instance_type = "t2.micro"
}
151. What is the purpose of the terraform state mv command?
terraform state mv moves a resource in the state file to a new address, useful for renaming
resources without recreating them.
Example:
terraform state mv aws_instance.old_name aws_instance.new_name
152. What is a backend in Terraform?
A backend in Terraform defines where and how state is loaded and stored. It can be local or
remote (e.g., S3, Consul, etc.).
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
 
154. What is the difference between local and remote backends in
Terraform?
Local backends store the state file on the local filesystem, while remote backends store the state
file in a remote storage service (e.g., S3, Consul).
Example (local backend):
terraform {
backend "local" {
path = "terraform.tfstate"
}
}
Example (remote backend):
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
155. How do you manage module versioning in Terraform?
Module versioning in Terraform can be managed using the version argument in
the source attribute of a module block, typically in combination with a registry.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
156. What is the Terraform Registry?
The Terraform Registry is a public repository of Terraform modules and providers that can be
used to discover and use pre-built modules and providers.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
157. How do you generate and view a resource graph in Terraform?
A resource graph can be generated using the terraform graph command and can be viewed
using tools like Graphviz.
Example:
terraform graph | dot -Tpng > graph.png
158. What is the purpose of the terraform validate command?
terraform validate is used to validate the syntax and configuration of the Terraform files
without creating any resources.
Example:
terraform validate
159. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
160. What are locals in Terraform and how do you use them?
Locals in Terraform are used to define local values that can be reused within a module. They help
avoid repetition and make configurations more readable.
Example:
locals {
instance_type = "t2.micro"
ami_id = "ami-0c55b159cbfafe1f0"
}
resource "aws_instance" "example" {
ami = local.ami_id
instance_type = local.instance_type
}
161. How do you handle provider dependencies in Terraform?
Provider dependencies in Terraform are managed using the required_providers block in
the terraform block, specifying the version constraints.
Example:
terraform {
required_providers {
aws = {
source = "hashicorp/aws"
version = "~> 3.0"
}
}
}
162. What is the terraform apply command used for?
terraform apply applies the changes required to reach the desired state of the configuration.
It executes the plan created by terraform plan.
Example:
terraform apply
163. What is the terraform destroy command used for?
terraform destroy is used to destroy the infrastructure managed by Terraform. It removes all
the resources defined in the configuration.
Example:
terraform destroy
164. What are output values in Terraform and how are they used?
Output values are used to extract information from the resources and make it accessible after
the apply phase. They can be used to output resource attributes.
Example:
output "instance_id" {
value = aws_instance.example.id
}
165. How do you manage different environments (e.g., dev, prod) in
Terraform?
Different environments can be managed using workspaces or separate directories with different
variable files and state files.
Example:
terraform workspace new dev
terraform workspace new prod
166. What is the difference between count and for_each in Terraform?
count is used to create multiple instances of a resource, while for_each is used to iterate over a
map or set of values to create multiple instances.
Example (count):
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
Example (for_each):
resource "aws_instance" "example" {
for_each = toset(["instance1", "instance2"])
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
167. How do you use loops in Terraform?
Loops in Terraform can be implemented using the count and for_each meta-arguments, as
well as the for expression in variable assignments.
Example:
variable "instance_names" {
type = list(string)
default = ["instance1", "instance2"]
}
resource "
aws_instance" "example" {
for_each = toset(var.instance_names)
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
168. What is a count parameter in Terraform?
The count parameter in Terraform is used to create multiple instances of a resource based on a
specified number.
Example:
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
169. What are Terraform Cloud and Terraform Enterprise?
Terraform Cloud and Terraform Enterprise are commercial versions of Terraform that provide
collaboration, governance, and automation features.
Example:
terraform {
backend "remote" {
organization = "my-org"
workspaces {
name = "my-workspace"
}
}
}
170. How do you use a Terraform backend?
A backend is configured using the terraform block in the configuration file, specifying the
backend type and its configuration.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
171. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
 
173. What is the purpose of the terraform workspace command?
The terraform workspace command is used to create, select, and manage multiple
workspaces, allowing different states to be associated with the same configuration.
Example:
terraform workspace new dev
terraform workspace select dev
 
resource "aws_secretsmanager_secret_version" "example" {
secret_id = aws_secretsmanager_secret.example.id
secret_string = jsonencode({
username = "example_user"
password = "example_password"
})
}
175. What is the terraform console command used for?
terraform console opens an interactive console for evaluating expressions, testing
interpolation syntax, and debugging configurations.
Example:
terraform console
176. How do you reference data sources in Terraform?
Data sources are referenced using the data block and can be used to fetch information about
existing infrastructure or services.
Example:
data "aws_ami" "example" {
most_recent = true
owners = ["amazon"]
filter {
name = "name"
values = ["amzn-ami-hvm-*"]
}
}
resource "aws_instance" "example" {
ami = data.aws_ami.example.id
instance_type = "t2.micro"
}
177. What is the purpose of the terraform state mv command?
terraform state mv moves a resource in the state file to a new address, useful for renaming
resources without recreating them.
Example:
terraform state mv aws_instance.old_name aws_instance.new_name
178. What is a backend in Terraform?
A backend in Terraform defines where and how state is loaded and stored. It can be local or
remote (e.g., S3, Consul, etc.).
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
179. How do you secure the state file in Terraform?
State files can be secured by storing them in remote backends with proper access controls and
encryption, such as AWS S3 with server-side encryption and access control policies.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
encrypt = true
}
}
180. What is the difference between local and remote backends in
Terraform?
Local backends store the state file on the local filesystem, while remote backends store the state
file in a remote storage service (e.g., S3, Consul).
Example (local backend):
terraform {
backend "local" {
path = "terraform.tfstate"
}
}
Example (remote backend):
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
181. How do you manage module versioning in Terraform?
Module versioning in Terraform can be managed using the version argument in
the source attribute of a module block, typically in combination with a registry.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
182. What is the Terraform Registry?
The Terraform Registry is a public repository of Terraform modules and providers that can be
used to discover and use pre-built modules and providers.
Example:
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
version = "2.0.0"
}
183. How do you generate and view a resource graph in Terraform?
A resource graph can be generated using the terraform graph command and can be viewed
using tools like Graphviz.
Example:
terraform graph | dot -Tpng > graph.png
184. What is the purpose of the terraform validate command?
terraform validate is used to validate the syntax and configuration of the Terraform files
without creating any resources.
Example:
terraform validate
185. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
186. What are locals in Terraform and how do you use them?
Locals in Terraform are used to define local values that can be reused within a module. They help
avoid repetition and make configurations more readable.
Example:
locals {
instance_type = "t2.micro"
ami_id = "ami-0c55b159cbfafe1f0"
}
resource "aws_instance" "example" {
ami = local.ami_id
instance_type = local.instance_type
}
187. How do you handle provider dependencies in Terraform?
Provider dependencies in Terraform are managed using the required_providers block in
the terraform block, specifying the version constraints.
Example:
terraform {
required_providers {
aws = {
source = "hashicorp/aws"
version = "~> 3.0"
}
}
}
188. What is the terraform apply command used for?
terraform apply applies the changes required to reach the desired state of the configuration.
It executes the plan created by terraform plan.
Example:
terraform apply
189. What is the terraform destroy command used for?
terraform destroy is used to destroy the infrastructure managed by Terraform. It removes all
the resources defined in the configuration.
Example:
terraform destroy
190. What are output values in Terraform and how are they used?
Output values are used to extract information from the resources and make it accessible after
the apply phase. They can be used to output resource attributes.
Example:
output "instance_id" {
value = aws_instance.example.id
}
191. How do you manage different environments (e.g., dev, prod) in
Terraform?
Different environments can be managed using workspaces or separate directories with different
variable files and state files.
Example:
terraform workspace new dev
terraform workspace new prod
192. What is the difference between count and for_each in Terraform?
count is used to create multiple instances of a resource, while for_each is used to iterate over a
map or set of values to create multiple instances.
Example (count):
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
Example (for
_each):
resource "aws_instance" "example" {
for_each = toset(["instance1", "instance2"])
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
193. How do you use loops in Terraform?
Loops in Terraform can be implemented using the count and for_each meta-arguments, as
well as the for expression in variable assignments.
Example:
variable "instance_names" {
type = list(string)
default = ["instance1", "instance2"]
}
resource "aws_instance" "example" {
for_each = toset(var.instance_names)
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
tags = {
Name = each.key
}
}
194. What is a count parameter in Terraform?
The count parameter in Terraform is used to create multiple instances of a resource based on a
specified number.
Example:
resource "aws_instance" "example" {
count = 3
ami = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
}
195. What are Terraform Cloud and Terraform Enterprise?
Terraform Cloud and Terraform Enterprise are commercial versions of Terraform that provide
collaboration, governance, and automation features.
Example:
terraform {
backend "remote" {
organization = "my-org"
workspaces {
name = "my-workspace"
}
}
}
196. How do you use a Terraform backend?
A backend is configured using the terraform block in the configuration file, specifying the
backend type and its configuration.
Example:
terraform {
backend "s3" {
bucket = "my-terraform-state"
key = "global/s3/terraform.tfstate"
region = "us-west-2"
}
}
197. What is the terraform fmt command used for?
terraform fmt formats the configuration files to follow the Terraform style guide, making the
code consistent and readable.
Example:
terraform fmt
198. How do you use a lock file in Terraform?
A lock file (.terraform.lock.hcl) is used to lock provider versions, ensuring consistency in
provider versions across different environments.
Example:
terraform init
199. What is the purpose of the terraform workspace command?
The terraform workspace command is used to create, select, and manage multiple
workspaces, allowing different states to be associated with the same configuration.
Example:
terraform workspace new dev
terraform workspace select dev
