---
layout: post
title: "Setting Up AWS IAM Users, Roles, and MFA for CLI Access"
date: 2024-11-01
categories: AWS IAM AWSCLI Security
---

In this post, we'll walk through setting up a restricted AWS user, creating an IAM role with permissions for resource creation, assigning the role to the user, configuring the AWS CLI, and optionally enabling MFA for secure access.

## 1. Setting up a User with No Permissions

To create a user without any permissions:

1. Open the AWS Management Console and go to **IAM**.
2. Select **Users** from the left-hand menu, then click **Add user**.
3. Provide a username, for example, `developer`, and select **Programmatic access** to enable CLI/API access.
4. Skip the permissions step, leaving the user without any permissions.
5. Review and create the user.
6. Open the `developer` user. On the summary select the **Create access key**.
7. For the use-case select **Command Line Interface (CLI)**, click next to create the access key.
8. Make sure to note down the **Access Key ID** and **Secret Access Key**.

This user has no permissions initially and will require a role assignment to perform any actions.

## 2. Creating an IAM Role for Resource Creation

Now, let’s create an IAM role that grants specific permissions for resource creation:

1. In the IAM console, select **Roles** from the left-hand menu, then click **Create role**.
2. Choose **AWS account** as the trusted entity type and select **This account** as the use case.
3. Attach a policy to allow resource creation. For example, you could use the **PowerUserAccess** policy for broad permissions or create a custom policy with only the necessary permissions.
    - To create a custom policy, click on **Create policy**, define the permissions (e.g., allowing actions like `ec2:RunInstances`), and attach it to the role.
4. Name the role (e.g., `ResourceCreatorRole`) and complete the setup.

## 3. Assign the Role to the User and Configure for AWS CLI

With the role created, we can now set up the user to assume this role via the AWS CLI.

### Attach a Policy for Role Assumption

To enable role assumption, the user needs a policy allowing them to assume the role:

1. Go to the **Policies** section and create a new inline policy for the user.
2. Use the following policy JSON, replacing `arn:aws:iam::ACCOUNT_ID:role/ResourceCreatorRole` with your role's ARN:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "sts:AssumeRole",
          "Resource": "arn:aws:iam::ACCOUNT_ID:role/ResourceCreatorRole"
        }
      ]
    }
    ```
3. Name the policy `AssumeResourceCreatorRole`.
4. Go to the `developer` user IAM account. On permissions attach the `AssumeResourceCreatorRole` policy.

### AWS CLI Configuration

1. Configure the user’s AWS profile:
    ```bash
    aws configure --profile developer
    ```
2. Use the aws sts assume-role command, specifying the --serial-number and --token-code parameters:
    ```bash
    aws sts assume-role \
    --role-arn arn:aws:iam::ACCOUNT_ID:role/ResourceCreatorRole \
    --role-session-name developer-session \
    --profile developer
    ```
3. This will issue temporary credentials and the `developer` can assume the `ResourceCreatorRole`.

## 4. (Optional) Enabling MFA for Extra Security

Adding MFA for the user enhances security. Here’s how to set it up:

1. In the **IAM** console, go to **Users** and select your user.
2. Under the **Security credentials** tab, click **Manage MFA device** and set up an MFA device (either virtual or hardware).
3. Update the IAM role `ResourceCreatorRole` Trust relationships policy to enforce MFA:
    
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::ACCOUNT_ID:user/developer"
                },
                "Action": "sts:AssumeRole",
                "Condition": {
                    "Bool": {
                        "aws:MultiFactorAuthPresent": "true"
                    }
                }
            }
        ]
    }
    ```

4. To obtain session tokens with MFA, the user needs to run:
    ```bash
     aws sts assume-role \
    --role-arn arn:aws:iam::ACCOUNT_ID:role/ResourceCreatorRole \
    --role-session-name with-token-session \
    --serial-number arn:aws:iam::ACCOUNT_ID:mfa/developer \
    --profile developer \
    --token-code 123456
    ```

With these steps, you've set up a secure user, an IAM role with limited permissions, configured CLI access, and enabled optional MFA for enhanced security.
