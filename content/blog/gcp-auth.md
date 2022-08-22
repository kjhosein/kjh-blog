---
title: "Authentication of Users & Apps to Google Cloud for Local Workstation Development"
date: 2022-02-22T17:45:18-04:00
draft: false
---

If you are developing code (on your workstation) to interact with the Google Cloud Platform (GCP) using either the Google Cloud SDK Command Line Tools (i.e. gcloud) and/or the Google Cloud Client Libraries, then you'll need to authenticate first.

This article was inspired by the excellent explanatory [how-to article created by John Tucker](https://codeburst.io/google-cloud-authentication-by-example-1481b02292e4), and is essentially a 'cheat-sheet' version for that how-to.

> Note that this article does not cover how to authenticate when you or your proxy is inside GCP or in another environment.

## Two (2) Sets of Tools

The first very important thing to note is that there these 2 sets of tools are authenticated separately.

1. Google Cloud SDK Command Line Tools:
The gcloud CLI manages authentication, local configuration, developer workflow, interactions with Google Cloud APIs.

2. Google Cloud Client Libraries are the recommended language-specific libraries for calling Google Cloud APIs.

## Two (2) Types of Accounts

There are two (2) different types of accounts that can be authenticated: User and Service Accounts.

1. User accounts are managed as Google Accounts, and they represent a developer, administrator, or any other person who interacts with Google Cloud. They are intended for scenarios where your application needs to access resources on behalf of a human user.

2. Service accounts are managed by IAM, and they represent non-human users. They are intended for scenarios where your application needs to access resources or perform actions on its own, such as running App Engine apps or interacting with Compute Engine instances.

### Impersonation

Another possible wrench option is where one account can impersonate another. E.g., a user impersonating a Service Account or even one
service account impersonating another service account.

## Scenarios

There are different scenarios where you may need to use some combination of authentication for either _gcloud_ or Cloud Client Libraries with either a user or service account.

### Scenario #1: Authenticating as a user in order to use gcloud tools

```
$ gcloud auth login
```

### Scenario #2: Authenticating as a user in order to use client libraries

```
$ gcloud auth application-default login [--project <project_name>]
```

### Scenario #3: Authenticating as a Service Account in order to use gcloud tools

First create and download a key for the service account (JSON format). Note: for security reasons, your organization may have set a policy to prevent this behavior.

```
$ gcloud auth activate-service-account <svc_acct_email_id> --key-file=<svc_acct_key.json>
```

### Scenario #4: Authenticating as a Service Account in order to use client libraries

To authenticate as a service account, set the environment variable, `GOOGLE_APPLICATION_CREDENTIALS`, with its value set to the path to the downloaded JSON file.

```
 $ export GOOGLE_APPLICATION_CREDENTIALS=/home/user/documents/<svc_acct_key.json>
```

### Scenario #5: User impersonating a Service Account in order to use gcloud tools

First authenticate as the user (like in Scenario #1), then use the auth/impersonate_service_account configuration setting.

```
$ gcloud auth login
$ gcloud config set auth/impersonate_service_account <svc_acct_email_id>
```

### Scenario #6: User impersonating a Service Account in order to use client libraries

Ensure that `GOOGLE_APPLICATION_CREDENTIALS` is not set, and that there is no user account auth:

```
$ unset GOOGLE_APPLICATION_CREDENTIALS
$ gcloud auth application-default revoke
```

Create a short-lived Access Token:

```export ACCESS_TOKEN=$(gcloud auth print-access-token --impersonate-service-account <svc_acct_email_id>)```

Finally, in your code, you will need to reference the `ACCESS_TOKEN` and use that in creating your OAuth2 credentials parameter. Here is some [sample Python code](https://gist.github.com/larkintuckerllc/d7d885301085c77cfd0fb33715bca67f#file-gc_sdk-py) that is listing the storage buckets:

{{< gist larkintuckerllc d7d885301085c77cfd0fb33715bca67f >}}

## References


* Very thorough tutorial on authenticating to GCP w/ user and svc accounts:  
    https://codeburst.io/google-cloud-authentication-by-example-1481b02292e
* gcloud auth application-default login  
    https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login
* How to create short-lived Service Account credentials:  
    https://cloud.google.com/iam/docs/creating-short-lived-service-account-credentials
* IAM Permissions Reference (searchable):  
    https://cloud.google.com/iam/docs/permissions-reference


## Related

A role is grantable on or above a resource if it contains any permissions for that resource type. [More details](https://cloud.google.com/iam/docs/viewing-grantable-roles#iam-grantable-roles-console) in the GCP docs.
