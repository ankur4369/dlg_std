# JFrog

In DL [JFrog](https://directline.jfrog.io/) provides our artifact management,
secret detection, vulnerability scanning for detecting issues in dependencies
and artifacts used in infrastructure and applications.

Many of JFrogs security capabilities can be integrated into the development
lifecycle. To understand more see the [Advanced Security](./04-advanced-security.md)
section.

JFrog is managed by the Technology Tooling squad. Detailed information can
be found in the Confluence page [here](https://directline.atlassian.net/wiki/spaces/STS/pages/13155402439/JFrog+Artifactory+Enterprise+X-Ray).

:::note
Artifactory’s billing is based on the amount of storage we consume across the
platform. Repositories that take up more than a few Gb for snapshots or
oversized cache’s, will be chased for clean up.

Snapshots should all be stored in the same file in order for Xray to identify
them and remove older versions.
:::

## Key Features

1. **Repository Manager**

   - Stores, manages, and distributes binary artifacts (e.g., Docker images, npm
     packages, JAR files).
   - Supports multiple package
     formats: Maven, npm, PyPI, Docker, Helm, etc. Integration with CI/CD tools like
     Jenkins and GitHub Actions.

1. **Security & Compliance Scanning**

   - JFrog provides Xray for deep security scanning for vulnerabilities in
     artifacts and dependencies.
   - Supports license compliance management to prevent legal issues.
   - Integrates with Artifactory for real-time threat detection.

1. **Distribution (Software Release Management)**

   - Securely distributes software updates and releases.
   - Ensures immutable releases, preventing unauthorised modifications.

1. **JFrog Access (Identity & Access Management)**

   - Role-based access control (RBAC) for secure artifact management.
   - Supports SSO (Single Sign-On) and API key-based authentication.

## Access

Access is managed through AD Single Sign-on. A request can be made through the
[Manage my Business System Access](https://dlg.service-now.com/sp?id=sc_cat_item_guide&table=sc_cat_item&sys_id=691939fedb8610909caebd5c689619a8)
form in ServiceNow. Requesting business system `JFrog Artifactory Enterprise`
and your required roles should be `JFRO: User` unless in the security team.

This provides basic permissions on first login. You should then contact
[Project Admins](https://directline.atlassian.net/wiki/spaces/STS/pages/13240074852)
for them to grant you access to repositories.

### Login

To login to the JFrog Artifactory platform, go to [https://directline.jfrog.io/](https://directline.jfrog.io/).

You will need to use the `SAML SSO` button to login via SSO.

![login](./assets/login.png)

### Service Accounts

Service Accounts for automation in CI/CD are available but are linked to an SSO
account. Please see this Confluence [guide](https://directline.atlassian.net/wiki/spaces/STS/pages/13272286079/Automation+Accounts)
to request one.

:::note
JFrog now supports OIDC from GitHub. Please see [here](https://github.com/marketplace/actions/setup-jfrog-cli#authentication-methods).
In the future this guide will be updated to OIDC as the preferred method. Service
Accounts will no longer be supported by the Technology Tooling squad.
:::

## GitHub Actions

To interact with JFrog in CI/CD we can use the [setup-jfrog-cli](https://github.com/marketplace/actions/setup-jfrog-cli)
GitHub Action. A simple example is provided below:

```yaml
- uses: jfrog/setup-jfrog-cli@v4
  env:
    JF_URL: "https://directline.jfrog.io"
    JF_PROJECT: "project-key"

    # Basic authentication credentials
    JF_USER: ${{ secrets.JF_USER }}
    JF_PASSWORD: ${{ secrets.JF_PASSWORD }}
    # or
    # JFrog Platform access token
    JF_ACCESS_TOKEN: ${{ secrets.JF_ACCESS_TOKEN }}

- run: jf --version
```

:::note
JFrog supports OIDC integration with GitHub. Therefore basic authentication
through service accounts is not recommended. To understand the setup
required please see section [JFrog OIDC](../../security/oidc/04-jfrog.md).
:::

For further details on the GitHub Action see the cli documentation.
