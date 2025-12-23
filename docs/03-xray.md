# Xray

Please see JFrog [docs](https://www.jfrog.com/confluence/display/JFROG/Xray+Security+and+Compliance).

## What does it do?

JFrog Xray scans your artifacts, builds and Release Bundles and detects security
vulnerabilities and licenses in software components.

## Handling Critical Vulnerabilities

If you are seeing an error message `Artifact download request rejected:`
`<package-name> was not downloaded due to the download blocking policy`
`configured in Xray for <repository>.` Then you should follow these steps to
resolve.

:::note
An `403 (Forbidden)` HTTP error code will indicate you were not allowed to access
this artefact. Where an authentication issue occurs, `401 (Unauthorized)` would
be returned.

Navigate to UI. You will see a list of repositories under `Artifactory → Artifacts`.
Locate the repository storing your dependency/package. For remote artifacts you
must search under `<repository-name>-cache`, however we cannot ignore violations
from remote repositories or their cache.

![artifacts](./assets/artifacts.png)

Once you have located your package you will be greeted by the following tabs
including Xray. Under `Xray → Watched Violations` you will be
able to see current active violations **OR** ignored violations for packages
with existing ignore rules set up for each project.

![watch](./assets/watch-violations.png)

:::note
Please note that if you cannot see the Xray tab on a given artifact:

- The artifact type may not support Xray scanning

- On artifacts in remote repositories, there is no Xray tab (this is not true
  for their corresponding cache though)

  :::

Clicking on the Violations ID will provide additional information including the
full CVE or Xray number, Fixed versions , CVE score and vulnerable component -
among other information.

![list](./assets/violation-list.png)

![violation](./assets/violation.png)

Users can use this information to amend Artifacts or build mitigation to prevent
security issues. For additional support users should contact the security team
to assist and develop a plan to protect against security issues.

:::note
This applies to both Prod and Non-prod scenarios - it is the team's
responsibility to manage their code and ensure its safe for use - not
Technology Tooling.
:::

## Patching Vulnerabilities

It is preferred for security reasons for teams to patch the version of the
affected package to remediate the vulnerability, or where it is not possible,
move away from the package.

For development work, if this is not possible, there is likely a need for an
ignore rule to be implemented temporarily to continue testing and development.
In this event, please ensure you have a plan in place to remove vulnerabilities
before moving to production.

For Instances where no patch can be applied and is needed for production
environments, then mitigations should be in place before an ignore rule is
applied.

## Applying for a Security Exception

If there is no possibility of remediating the vulnerabilities violations by
upgrade or transferring away from the affected package or library, then a
Security Exception is required in order to ignore the vulnerability and remove
the block on the package.

After locating the affected dependency you can export violations:

![scan-data](./assets/scan-data.png)

![export-scan-data](./assets/export-scan-data.png)

The export is integral for the next step, so that Security can be properly
informed to approve an exception and the Tooling Team can implement the
required violation Ignore Rules when that exception is approved.

:::warning
Security will only approve exceptions based on a list of CVEs and how they are
implemented in a given system/Project to determine their risk

Tooling will apply Ignore Rules based on a one-to-one mapping of 1 CVE → 1
Component → 1 Ignore Rule. This is independent of artifact path.
:::

Utilising the above information, use the following form to request a
[Security Exception Review](https://dlg.service-now.com/sp?id=sc_cat_item&sys_id=cdfa8315db45d810bcdba155059619c8).

:::note
Please include a tabular view of unique vulnerability (CVE) to its corresponding
impacted component and version, so that security can review easily and Tooling
can apply the Ignore Rules efficiently.
:::

:::warning
When submitting a Security Exception Review request, do not enter JFrog for the
question “Please select the Application or Service name”. This request process
triggers an approval for the Technical SME of the application provided in that
field.

The Technical SME for JFrog is in no position to approve your Exception request
as they have no understanding of the environment the exception is for. If you do
this, the request will be rejected at this stage and you will have to resubmit
it with the correct application.
:::

You will also need to provide grounds for the exception, including one or
multiple of the following:

1. No fixed version exists (usually only true for zero-day vulnerabilities
   and/or abandoned packages). You need to demonstrate this is the case, e.g. by
   reference to the NIST CVE page for the vulnerability.
1. A fixed version exists but you are unable to upgrade to it for a functional
   reason. You need to demonstrate this, e.g. by build output or functional test
   output.
1. You are able to upgrade but there are pressing business reasons why you
   cannot do so immediately. In this case, request a short term security exception.
   You need to document the business case and your remediation plan, with a
   committed timeframe.
1. There is no fix because the package is no longer maintained. Your exception
   request should include a remediation plan to move off the unmaintained package
   to an alternative package.
1. The package maintainer `will not fix`. Your exception request will need to
   include a mitigation plan. Typically when a package maintainer `will not fix`
   it is because the vulnerability can be prevented by secure configuration - if
   so, include this in your mitigation plan.

## Tooling Team to Apply the Ignore Rule(s) to the Violations

Once the above submitted Security Exception has been approved by security, the
Technology Tooling team will need to apply the Ignore Rules to the affected
Project. Please submit a request via this form [Technology Tooling Work Engagement](https://dlg.service-now.com/sp?id=sc_cat_item&table=sc_cat_item&sys_id=c26de5709782719045c27a671153afb4)
.

:::note
For the question “Which statement best represents your request?” on the form,
you should select “A change to something that exists today”.
:::

- You must include the approved security exception request reference for
  reference.
- You must include the referenced CVE numbers and affected components in a
  readable, tabular view.
- You must include which Project (usually identified by repository paths) is in
  scope for the Ignore Rule(s).

:::note
The Tooling Team cannot apply ignore rules to violations in shared remote
repositories, the artifact needs to move to a singular Project’s repository
(e.g. local) before anything can be done about them. This is true for the cache
of the remote repository also. See here for more information on how to do this.

This is because the exception may be approved due to the context of use for one
platform using the shared resources, but not others where the risk appetite may
be different due to how it is deployed.
:::

:::danger
Usually these requests are complete within the three-day SLA for Tooling
Requests, but for large numbers of vulnerabilities and components the expected
delivery time is longer than the three-day SLA for standard requests, and it may
be converted to a Story which follows a prioritisation process
without SLAs.
:::

## Expired Exceptions & Ignore Rules

Ignore Rules are being configured in JFrog to expire at the same time as the
expiry date requested on the associated Security Exception Review request.

To request extension or renewal of Security Exception Review Requests and then
renewal of associated Ignore Rules in JFrog by tooling, please follow the same
process as requesting a new one, as above. You can cite the previous Request
reference on the Security Exception Review form to aid things.

Please do so with enough advance warning for both requests so as not to be
disappointed if Ignore Rules expire and artifacts become blocked for download
again via Xray, before renewed exceptions can be put in place.

::note
It is the responsibility of application teams to monitor the expiry of
Ignore Rules / Exception Requests for their platform, not Technology Tooling’s
or Security Engineering.
:::
