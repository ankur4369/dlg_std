# SonarQube Cloud

SonarQube Cloud is our SaaS instance of Sonar which is managed by the
[Tooling Squad](https://directline.atlassian.net/wiki/spaces/STS/pages/330629201/SonarCloud)
.

## Access

SonarQube Cloud can be accessed from [here](https://sonarcloud.io/organizations/direct-line-group/projects)
.

Access is managed through use of your GitHub credentials which are linked to
the DL GitHub organisation.

![login](./assets/cloud/login.png)

Once logged in you should now see a list of current projects in DL.

![ui](./assets/cloud/ui.png)

If you don't see any projects a quick way of switching to the correct view is
to select your profile from the top right of the window and selecting
`Direct Line Group`:

![profile](./assets/cloud/profile.png)

## New Projects

To setup a new project for your repository this needs to be manually completed
in the SonarQube Cloud UI.

Select the `+` top right next to your profile picture:

![newproject](./assets/cloud/newproject.png)

Then use the UI to find your GitHub repository:

![projects](./assets/cloud/projects.png)

Once setup you need to disable automatic analysis:

![analysis](./assets/cloud/analysis.png)

This can be configured by selecting Administration → Analysis Method and
disabling ‘Automatic Analysis’.

Without this the standard Claims Digital integrations will fail as CI and
Automatic Analysis cannot be both enabled concurrently.

Users which can make this change must have admin permissions on the source
repository as permissions are synchronised to SonarQube Cloud.

## Quality Gates

Quality gates can be configured, the below example shows the default Sonar Way
gate:

![qualitygate](./assets/cloud/qualitygate.png)

## Add Configurations to Project

1. Create a configuration file in your project's root directory called
   `sonar-project.properties`.

   ```text
   sonar.projectKey=Direct-Line-Group_<your_repo_name>
   sonar.organization=direct-line-group
   sonar.host.url=https://sonarcloud.io

   # This is the name and version displayed in the SonarCloud UI.
   sonar.projectName=<your_repo_name>
   # Path is relative to the sonar-project.properties file
   sonar.sources=<path_to_source_code>
   ```

   For other configurations to consider please see the Sonar docs [here](https://docs.sonarqube.org/latest/analysis/analysis-parameters)
   .

   - `sonar.profile`
   - `sonar.sources`
   - `sonar.coverage.exclusions`
   - `sonar.core.codeCoveragePlugin`
   - `sonar.jacoco.reportPath`

1. Running locally. We can configure the project to scan the code locally. The
   `SonarScanner` supports multiple languages, e.g.
   [Maven](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner-for-maven/)
   , [Gradle](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner-for-gradle/)
   , [.NET](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/dotnet/introduction/)
   , [NPM](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/npm/introduction/).
   See the links for more information.

   In this example we will add the Maven SonarScanner plugin to a `pom.xml` for
   a Java project.

   The SonarScanner for Maven is recommended as the default scanner for Maven
   projects.

   The ability to execute the Sonar analysis via a regular Maven goal makes it
   available anywhere Maven is available (developer build, CI server, etc.)

   ```xml
   <build>
     <plugins>
         <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
         <plugin>
             <groupId>org.sonarsource.scanner.maven</groupId>
             <artifactId>sonar-maven-plugin</artifactId>
             <version>5.0.0.4389</version>
             <executions>
                 <execution>
                     <phase>verify</phase>
                     <goals>
                         <goal>sonar</goal>
                     </goals>
                 </execution>
             </executions>
         </plugin>
     </plugins>
   </build>
   ```

1. Install SonarLint plugin for your IDE to analyse as you develop.

   - [IntelliJ](https://plugins.jetbrains.com/plugin/7973-sonarlint)
   - [VS Code](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarlint-vscode)

## SonarQube Cloud Badges in GitHub Repos

You can add SonarCloud badges to the project `README` to give code quality
metrics against specific branches.

1. From within your project screen on SonarQube Cloud select the information
   tab on the left-hand side.
1. Select the metric that you wish to appear in on your project `README`.
1. Click the copy button which will copy a line in markdown that can be added
   to your `README`.

![badges](./assets/cloud/badges.png)

## GitHub Actions

SonarQube Cloud analysis can be run from our CI pipelines using GitHub Actions.

In the below example for Maven we add these steps to the workflow file:

```yaml
- name: Cache SonarQube packages
  uses: actions/cache@v4
  with:
    path: ~/.sonar/cache
    key: ${{ runner.os }}-sonar
    restore-keys: ${{ runner.os }}-sonar

- name: Build with maven and analyze
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar
```

We can also utilise the [sonarcloud-github-action](https://github.com/SonarSource/sonarcloud-github-action)
but it is not possible for all languages. Please see the [do not use](https://github.com/SonarSource/sonarcloud-github-action#do-not-use-this-github-action-if-you-are-in-the-following-situations)
notes.

```yaml
- name: SonarCloud Scan
  uses: sonarsource/sonarcloud-github-action@master
  env:
    GITHUB_TOKEN: ${{ inputs.GITHUB_TOKEN }}
    SONAR_TOKEN: ${{ inputs.SONAR_TOKEN }}
```

:::note
The `SONAR_TOKEN` is automatically added to your repository on creation as
it is managed at the GitHub Organisation level.
:::

## Troubleshooting

### Permission Error

```text
Check ALM binding of project 'Direct-Line-Group_claims-digital-hub-ui-test'
Failed to check if project 'Direct-Line-Group_claims-digital-hub-ui-test' is bound
Detected project binding: ERROR
test: Could not find a default branch to fall back on. -> [Help 1]
```

This error is usually due to an access or set up problem inside SonarQube Cloud:

1. Check that the project key exists in SonarQube Cloud.
1. Ensure that Automatic Analysis is disabled as per the above guidance.
1. Ensure there is a secret configured inside the repository called SONAR_TOKEN.
1. Make sure that the SONAR_TOKEN secret is mapping to an environment variable
   of the same name.

```yaml
env:
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Other Issues

Any other issues please reach out via the ServiceNow
[Technology Tooling Work Engagement](https://dlg.service-now.com/sp?id=sc_cat_item&table=sc_cat_item&sys_id=c26de5709782719045c27a671153afb4)
.

:::warning
The Tooling squad manage licence volumes in SonarQube Cloud. Therefore,
if no code changes are made to a project for 90 days the project is removed.
Please see the Confluence [page](https://directline.atlassian.net/wiki/spaces/STS/pages/13238240552/Housekeeping+Inactive+Projects)
for more details.

Also when a project is removed your fancy GitHub badges will fail 😢
:::
