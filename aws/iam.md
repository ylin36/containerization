# 0. Use IAM instead of root when possible
Setup MFA, and create IAM for typical usages instead.

# 1. Users
Specific individuals, can receive personal logins
# 2. Groups
Collection of users. maps to policies / permissions
# 3. Roles (NOT TIED DIRECTLY TO USERS)
Collection of policies

*you can't assign roles to users*

Role is a way to provide permissions to someone (a customer, supplier, contractor, employee, an EC2 instance, some external application outside AWS trying to consume your services) without creating a user for it.

# 4. Policies/Permissions
Allow/Deny low level permissions to aws resources
