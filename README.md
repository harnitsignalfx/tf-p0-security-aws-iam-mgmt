# Prerequisites
    TerraformExecutionRole

If you need to create this role:
1. Modify the file iam/terraform-execution-role.json and add your AWS Principal (Role) that will be used to assume the terraform role.
2. Execute the following steps in the root folder:

```
aws iam create-role \
    --role-name TerraformExecutionRole \
    --assume-role-policy-document file://iam/terraform-execution-role.json

aws iam put-role-policy \
    --role-name TerraformExecutionRole \
    --policy-name TerraformExecutionPolicy \
    --policy-document file://iam/terraform-execution-role-policy.json
```

# Steps:

1. Add your root account as a string value
2. Add your children/member accounts in an array of comma separated strings
3. Add the P0 Security Google Audience ID
4. Run the following commands:

```
terraform init
terraform plan
terraform apply
``` 


# Tasks performed: 
 
In the Root/Management Account:
    Created by TerraformExecutionRole:
        P0RoleIamManager role with Google federation
        P0RoleIamManagerPolicy (inline policy) attached to P0RoleIamManager
        role-creation-lambda-role for the Lambda function
        role-creation-policy (inline policy) for Lambda role
        Lambda function called create-member-account-roles




In Each Child Account:
    Created by Lambda function (which assumes OrganizationAccountAccessRole):   
        P0RoleIamManager role with same Google federation
        P0RoleIamManagerPolicy (inline policy) attached to P0RoleIamManager

# Workflow: 

1. Initial Setup:
Your AWS Identity → TerraformExecutionRole

Terraform uses your provided TerraformExecutionRole to create resources in root account


2. Lambda Creation:

Terraform creates a zip package containing:
    * Lambda function code (index.js)
    * Policy template (p0_role_policy.json)
Package is uploaded to AWS Lambda


3. Member Account Role Creation:
Lambda → OrganizationAccountAccessRole → Create P0RoleIamManager

Lambda function iterates through member accounts
For each account:
    1. Assumes the OrganizationAccountAccessRole
    2. Creates P0RoleIamManager with Google federation
    3. Reads policy template from package
    4. Replaces ${account_id} placeholder with current account ID
    5. Attaches policy to the role


4. Final Trust Chain for Users:
Google Federation (P0 Security) → P0RoleIamManager (in any account)

End users can assume P0RoleIamManager in any account through P0 Security
