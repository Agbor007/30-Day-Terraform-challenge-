# Day 7: Understanding Terraform State Part 2

## Participant Details
- **Name:** Duncan Gaturu
- **Task Completed:** : Workspace Layout 
- **Date and Time:** 2024-09-12 8:16pm


### Practice using Workspace Layout to manage terraform State
#### main.tf

```hcl
# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
 
}

#create s3 bucket
resource "aws_s3_bucket" "state" {
  bucket = "gaturu-state-bucket"
#Prevent accidental deletion of this S3 bucket
lifecycle {
  prevent_destroy = true
 }
}#/

#Enable versioning so you can see the full revision history of your
# state files
resource "aws_s3_bucket_versioning" "enabled" {
  bucket = aws_s3_bucket.state.id
  versioning_configuration {
     status = "Enabled"
 }
}

# Enable server-side encryption by default
resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
   bucket = aws_s3_bucket.state.id
rule {
  apply_server_side_encryption_by_default {
    sse_algorithm = "AES256"
   }
 }
}

# Explicitly block all public access to the S3 bucket
resource "aws_s3_bucket_public_access_block" "public_access" {
   bucket = aws_s3_bucket.state.id
   block_public_acls = true
   block_public_policy = true
   ignore_public_acls = true
   restrict_public_buckets = true
}

#create DynamoDB Table for Locking
resource "aws_dynamodb_table" "terraform_locks" {
   name = "terraform-running-locks"
   billing_mode = "PAY_PER_REQUEST"
   hash_key = "LockID"
   attribute {
     name = "LockID"
     type = "S"
 }
}
#Configure a backend for this Instance using the S3 bucket and DynamoDB table
terraform {
  backend "s3" {
# Replace this with your bucket name!
    bucket = "gaturu-state-bucket" 
    key = "workspaces-example/terraform.tfstate"
    region = "us-east-1"
# Replace this with your DynamoDB table name!
   dynamodb_table = "terraform-running-locks"
   encrypt = true
 }
}
output "s3_bucket_arn" {
  value = aws_s3_bucket.state.arn
  description = "The ARN of the S3 bucket"
}

output "dynamodb_table_name" {
   value = aws_dynamodb_table.terraform_locks.name
   description = "The name of the DynamoDB table"
}

```
