# SonarQube Server

SonarQube Server is self-managed instance specifically for languages not
supported by [CloudQube Cloud](02-cloud.md). E.g. `GoSu` for Guidewire with the
`GoQu` plugin.

Most languages are supported by the Cloud instance so please check before using
SonarQube Server.

## Access

[https://sonarqube-shared.sq.awsdlgctengineeringsharedtoolingprod.dlgctplatform.co.uk](https://sonarqube-shared.sq.awsdlgctengineeringsharedtoolingprod.dlgctplatform.co.uk)

Access is managed through AD Single Sign-on. A request can be made through the
[Manage my Business System Access](https://dlg.service-now.com/sp?id=sc_cat_item_guide&table=sc_cat_item&sys_id=691939fedb8610909caebd5c689619a8)
form in ServiceNow and requesting business system `Sonarqube (AWS Cloud)`.

Most users will be required to select the `SONQ: B4C User` role.

Start by navigating to the instance you wish to access. Enter your RACF ID and
password for Single Sign-on.

![login](./assets/server/login.png)

Once logged in you should now see a list of the current projects in DL.

![ui](./assets/server/ui.png)

## New Projects

New projects can only be added and configured by the SonarQube Server admins
before using as part of your repository CI checks in GitHub Actions.

Once your project is setup, add the SonarQube Server host URL and token provided
by the admins as secrets in your repository.

You can now add a scan step to your CI workflow. An example is provided below.

```yaml
- name: SonarQube scan with GoQu
  run: |
    sonar-scanner -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }}-Dsonar.login=${{ secrets.SONAR_TOKEN }}
```

To add a new project, request a token or raise issues, please reach out to the [admins](https://directline.atlassian.net/jira/software/c/projects/DEV/boards/6442)
.

## Quality Gates

The default quality gate in the SonarQube Server instances is `Sonar way`. Sonar
way contains several language profiles but it is the profile `gosu` which is
used for our Guidewire applications.

:::tip
If you do not require `Sonar way` and the profile `gosu` their is a good chance
you should be using the [Cloud](./02-cloud.md) instance.
:::
