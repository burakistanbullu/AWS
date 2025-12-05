# IAM General Information

What is IAM?

Identity and Access Management (IAM) is a service that controls authentication (who can access) and authorization (what they can do).

When creating a user, you provide authentication details, and by assigning groups and policies you handle the authorization part.

## Basic Concepts

### User

**What is a User in IAM?**

A user represents a real person who needs access to AWS.
When you create an IAM user, AWS provides a link like:
https://<-account-id->.signin.aws.amazon.com/console

Your AWS Account ID is part of this link (e.g., 554435643459) and when logging into the AWS console, you use ; Username, Password and that Account ID.

**Types of Users**

There are two types of users in AWS:

**1. Root User**
-   Created automatically when the AWS account is first set up.
-   Has unrestricted access to all AWS resources.
-   IAM policies cannot restrict the root user.
-   Security audits will highlight root login events as critical or unsafe.
-   Only one root user exists per AWS account.

**2. IAM User**

-   A regular user created by the root user or an admin.
-   Permissions are restricted using IAM policies.
-   Used for day-to-day operations.

**User Management Best Practices**

*AWS strongly recommends:*

“Use the root user only for account and administrative tasks.
Create IAM users for daily operational work.”

*In other words:*

-   Avoid logging in with the root user unless absolutely necessary.
-   Create an AdministratorAccess IAM user for admin operations (example: burak-admin).
-   Enable MFA (Multi-Factor Authentication) for the root user.
-   Avoid generating or using root access keys.
-   Do not use the root user for regular tasks.

### Policy,Group,Role
#### IAM Policy

-   Policies are JSON-based documents defining:
-   Who can do what actions under which conditions?
-   Policies can be attached to: Users, Groups, Roles

#### IAM Group

-   A group is a collection of users.
-   Instead of assigning policies to each user individually, you assign policies to a group and place users inside it.
-   This simplifies permission management, especially in large environments.

#### IAM Role

-   A role is used when a service, application, or external identity needs access to AWS resources.

- Roles do not have permanent credentials (no username/password).

- When an application assumes a role;
    
    -   AWS generates temporary security credentials (access key, secret key, session token).

    -   The application uses these credentials to interact with AWS securely.
