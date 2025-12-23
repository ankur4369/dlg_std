# Repositories

Repositories in JFrog are within Projects. Projects are managed by the
Technology Tooling squad. If you wish to create a new project please raise
a request through ServiceNow [here](https://dlg.service-now.com/sp?id=sc_cat_item&table=sc_cat_item&sys_id=c26de5709782719045c27a671153afb4)
and ensure you select `JFrog Artifactory Enterprise + X-Ray` as the System or
Service.

## Repositories

Repositories of a project can be listed by first selecting the project and then
`Artifacts`

![project-list](./assets/project-list.png)

You will then be presented with a list of repositories and the artifacts they
contain.

![repo-artifacts](./assets/repo-artifacts.png)

To manage the project repositories you will require administration access to the
project.

With administration permissions the menu option to administer the project will
be next to `Platform`.

![admin](./assets/admin.png)

Once selected the menu underneath the project selection should change to:

![menu](./assets/menu.png)

Select `Repositories` to display a table of those configured for the project.

![repos](./assets/repos.png)

### New Repositories

To create a new repository first select the `Create a Repository` button top
right.

![repo-create](./assets/repo-create.png)

You will now be presented with three options for the type of repository you
wish to create.

![repo-type](./assets/repo-type.png)

- Local
  - Stores artifacts created within our organisation.
  - Used for hosting internally developed binaries, Docker images, libraries,
    and dependencies.
  - Example: A Java team storing compiled JAR files in a Maven repository.
- Remote
  - Proxy and cache artifacts from external sources (e.g. Docker Hub, npm
    registry, Maven Central).
  - Reduces download time and ensures availability even if the external
    source is down.
  - Example: Needing dependencies from Maven Central can create a remote Maven
    repository in Artifactory and using Xray to scan.
- Virtual
  - Aggregates multiple local and remote repositories into a single logical
    repository.
  - Developers only need to pull artifacts from one endpoint instead of multiple
    sources.
  - Example: A virtual repository combining local, remote, and cached npm
    packages for easier access.

Once you have selected your type of repository, the type of packages held in the
repository must be chosen.

JFrog supports most common types. If you plan to store different types of packages
then select `Generic`.

![package-type](./assets/package-type.png)

Each package type of the repository can present different configuration options.
Please consult the JFrog [documentation](https://jfrog.com/help/r/jfrog-artifactory-documentation/repository-management)
to understand all the options.

The standard repository structure follows the following pattern for your own
team:

- [your team]-maven-dev/prod - Maven repo - where you team's jars will live.
- [your team]-pypi-dev/prod - Pypi repo - This is where you Python packages
  will live.
- all-remote - Virtual repository containing ALL project specific remote
  resources
  - Recommended **NOT** to use to use publicly available remote resources,
    E.g: Pypi. Create your own [project-key]-pypi

:::note
Each project should create a virtual repository (all-remote) to store all
remote resources.

Additionally each repository should be created as DEV or PROD, and named
accordingly. Development based repositories should ONLY hold up to 3 most
recent build artifact’s. Production will be required for a longer periods. For
more information see the [best practices](./05-best-practices.md)
:::

Once all fields are entered where required press the create button.

:::tip
In CI/CD your service account will require access to your project to pull and
push any artifacts. Ensure it has access under `User Management -> Members` and
the correct permissions set defined by a role.
:::

Any issues please raise a request with the Technology Tooling squad using the
ServiceNow link at the start of the page.

### Other Repositories

These repositories are shared across DL:

- dlg-ext-lib - Maven repo - this is exclusively for 3rd party libs which are
  not available in open source repositories, such as Oracle/SAS/GuideWire/etc.
  DO NOT publish packages available in jcenter / maven central, spring io on this
  repo.
- dlg-shared-lib - Maven repo - this is a repo to share jar libraries with the
  rest of DL. eg Some custom written SAS utility, AWS utilities, etc.
- dlg-shared-python - Pypi repo - this is a repo to share python packages with
  the rest of DL. eg Some custom written SAS utility, AWS utilities, etc.
- dlg-shared-generic - Binaries repo - binaries from 3rd party. eg terraform /
  gradle wrapper / maven wrapper
