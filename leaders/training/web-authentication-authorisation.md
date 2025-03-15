# Authentication and Authorisation

## Overview

In the digital age, the concepts of authentication and authorisation have become crucial, especially with the rise of web commerce. As more transactions and interactions move online, ensuring that users are who they claim to be (authentication) and that they have permission to access certain resources (authorisation) is vital for security and privacy.

## History and Context

The need for authentication and authorisation has grown alongside the internet. In the early days, simple password-based systems were sufficient. However, as web commerce expanded, so did the sophistication of cyber threats. High-profile incidents, such as the 2013 Target data breach and the 2017 Equifax breach, highlighted the devastating consequences of inadequate security measures. These breaches exposed millions of users' personal information, leading to financial losses and a loss of trust.

## Real-World Incidents

1. **Target Data Breach (2013)**: Hackers gained access to Target's network through a third-party vendor, compromising 40 million credit and debit card accounts.
2. **Equifax Breach (2017)**: A vulnerability in a web application led to the exposure of personal information of 147 million people, including Social Security numbers and birth dates.

## Common Authentication and Authorisation Processes

### Authentication

1. **Password-Based Authentication**: Users provide a username and password to gain access.
2. **Multi-Factor Authentication (MFA)**: Combines something the user knows (password) with something they have (a mobile device) or something they are (biometrics).
3. **OAuth**: Allows users to grant third-party applications access to their information without sharing passwords.

### Authorisation

1. **Role-Based Access Control (RBAC)**: Users are assigned roles that have specific permissions.
2. **Access Control Lists (ACLs)**: Define which users or system processes can access certain resources.
3. **JSON Web Tokens (JWT)**: Used to securely transmit information between parties as a JSON object.

## Example Implementations

### Password-Based Authentication

```python
def authenticate(username, password):
    user = get_user_by_username(username)
    if user and user.check_password(password):
        return True
    return False
```

### Multi-Factor Authentication (MFA)

```python
def send_otp(user):
    otp = generate_otp()
    send_sms(user.phone_number, otp)
    return otp

def verify_otp(user, otp):
    return user.otp == otp
```

### Role-Based Access Control (RBAC)

```python
def check_access(user, resource):
    if user.role in resource.allowed_roles:
        return True
    return False
```

Understanding and implementing robust authentication and authorisation mechanisms are essential for protecting users and maintaining trust in web-based systems. As threats evolve, so must the strategies to counter them, ensuring a secure online environment.

## Further reading

See the slides on [Network Security](https://docs.google.com/document/d/e/2PACX-1vR8uwUGveas_P60V6UvIZstKEQrsMs5mcsrw_U-CTOCOqSETTojjzslva7vlnOf7mIfi6tla-qPJ-mT/pub).
