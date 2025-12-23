# Repositories

Setting up a GitHub repository with good practices can greatly enhance
collaboration, security, and maintainability.

In DL these are our recommended practices for configuring your repository
settings.

:::tip
We have a script in the handbook repository`./scripts/github/create-repo.sh` to
create a new repository from these best practices. Clone the repository locally
to run it. Following the clone:

```sh
cd se-engineering-handbook/scripts/github
./create-repo.sh se-new-repo my-team
```

See the script for more details on the inputs.

Once the repository is created please continue to read this page as it
contains valuable information on ways of working such as how to handle releases.
:::

## Repository Settings

### Collaborators and teams

1. Use Teams from the Organisation not individuals for managing access.
1. Assign repository roles (read, write, admin) carefully based on team members
   responsibilities.
1. Use a `CODEOWNERS` file for:
   - showing clear ownership, and clarifying who to contact for questions or
     issues.
   - automated review requests, GitHub automatically requests reviews from the
     designated owners.
   - encouraging accountability, making it clear who is responsible for
     maintaining and reviewing.
   - improved code quality as owners are more familiar with content.

#### CODEOWNERS

When configuring your `CODEOWNERS` file it is best practice to:

1. Utilise path based expressions for allocating relevant SME's to your code
   reviews
1. If you utilise a monorepo - utilise expressions to ensure the right SME's are
   allocated to the appropriate areas of your repo based on skillset
1. Make use of a [GitHub
   team](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams)
   as the target for review of the given path within your repository; this
   allows easy extensibility of the reviewers
1. Lock down (at least part of (`CODEOWNERS`)) the `.github` folder of your
   repository to an administration group ensuring the controls you are building
   in cannot be easily overridden without a review from your senior team

##### CODEOWNERS Example

The below example gives an overview of a monorepo whereby multiple parts of the
stack for an application can have appropriate `CODEOWNERS` defined:

```markdown title="/.github/CODEOWNERS"
* @Direct-Line-Group/<platform>-admins
.github/** @Direct-Line-Group/<platform>-admins
backend/** @Direct-Line-Group/<platform>-backend
ui/** @Direct-Line-Group/<platform>-ui
infra/** @Direct-Line-Group/<platform>-devops
```

### Protections

Your repository should prevent unintended or malicious changes. Therefore
several protection methods can be set by using GitHub
[Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets).

Rulesets can be created in the repository settings under `Code and automation`
and `Rules` then `Rulesets -> New ruleset`.

#### Branches

At a minimum, your `main` or trunk branch should be protected by a branch
`Ruleset`. Create a new ruleset `New ruleset -> New branch ruleset`.

Give the ruleset the name of your branch which in this case should be `main`.
Next, configure the `Target branches`. If your `main` branch is your default you
can simply select `Include default branch`. If not you may have to add an
include/exclude pattern. Documentation on how to do this with the `fnmatch`
syntax can be found
[here](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository#choosing-which-branches-or-tags-to-target).

With the name of the ruleset and target branch set, ensure the following minimum
rules are set:

1. Require signed commits. To set this please follow this link to the the GitHub
   [docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)
   .
1. Require a pull request before merging. Therefore all changes are reviewed and
   checked.
   - Required approvals is >= 1.
   - Dismiss stale pull request approvals when new commits are pushed.
   - Require review from Code Owners.
   - Require approval of the most recent reviewable push.
   - Require conversation resolution before merging.
1. Require status checks to pass.
   - Require branches to be up to date before merging.
1. Block force pushes.

Now save the ruleset and ensure you activate it.

With the use of pull requests, it is also good practice to create a pull request
template to:

1. Provide a consistent format, making it easier for reviewers to understand the
   purpose and scope of the changes.
1. Prompts contributors to provide essential information, such as the rationale
   behind the changes, potential issues, context etc.
1. Checklists for code reviews, testing, or documentation updates, ensuring that
   contributors don’t overlook important steps.
1. A reminder of the contribution guidelines, reinforcing standards.

Pull request templates are named `pull_request_template.md` and placed into
`.github` folder. For more information see
[here](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository#adding-a-pull-request-template)

Further, pull request settings can also be found under
`Settings -> General -> Pull Requests`.

The following should be set:

1. Allow merge commits.
1. Allow squash merging.
1. Always suggest updating pull request branches.
1. Automatically delete head branches.

:::tip
Enabling the deletion of head branches will ensure no old branches are
left over which is just good housekeeping. If a problem is found post-merge,
just create it again and it ensures you have the latest code.
:::

#### Tags

Tag-based protection rules help prevent unauthorized or accidental deletion and
modification of tags, ensuring that important version markers remain intact and
immutable. You can also control who has the authority to create or update tags,
maintaining a clear and reliable release process.

We recommend that only admins or a subset of people in the repository can create
tags and releases.

##### Creation

Create a new ruleset `New ruleset -> New Tag ruleset`.

Name it e.g. `create-tag` and select the `Bypass list` role you wish to allow
creation of tags, then ensure `Target tag` equals `All tags` and the following
minimum rules are set:

1. Restrict creations
1. Block force pushes

Now save the ruleset and ensure you activate it.

##### Deletion

Create a new ruleset `New ruleset -> New Tag ruleset`.

Name it e.g. `delete-tag`, then ensure `Target tag` equals `All tags` and the
following minimum rules are set:

1. Restrict updates
1. Restrict deletions
1. Block force pushes

Now save the ruleset and ensure you activate it.

### Security

#### Dependabot

From GitHub:

:::info
Dependabot keeps your dependencies up to date by informing you of any
security vulnerabilities in your dependencies, and automatically opens pull
requests to upgrade your dependencies to the next available secure version when
a Dependabot alert is triggered, or to the latest version when a release is
published.
:::

Therefore Dependabot automatically helps you manage several dependencies which
can be manually time-consuming to keep up to date. It also provides an alerting
method when vulnerabilities are found.

To use Dependabot features navigate to `Settings -> Security -> Code Security`.

Ensure the following is enabled:

1. Dependency graph.
1. Dependabot.
   - Dependabot alerts.
   - Dependabot security updates.
   - Dependabot version updates. Learn more about configuring it
     [here](https://docs.github.com/code-security/dependabot/dependabot-version-updates/configuring-dependabot-version-updates)
   - Dependabot on Actions runners.

## Repository Content

### Readme

Each repository should have a detailed `README` file containing:

1. A clear overview of your project, including its purpose and functionality,
   making it easier for others to understand what it does.
1. Setup instructions, installation steps, and usage guidelines, helping users
   get started quickly.
1. Reference for contributors and users, detailing how to contribute, report
   issues, or seek help.

For more information on how to do this in GitHub please see
[here](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes#about-readmes).

### License

If applicable, a `LICENSE` file should be added which clarifies how others can
use your code and sets clear expectations regarding permissions and
restrictions.

For more information on how to do this in GitHub please see
[here](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/adding-a-license-to-a-repository).

## Release Management

GitHub
[Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases)
and tags should be used. This ensures you can create (and recreate) versioned
snapshots of your project, making tracking changes easier over time and
maintaining backward compatibility.

:::warning
Versioning software through branches is **discouraged** in DL. Using
branches is not immutable, they add extra complexities, greater overheads,
divergence from main code bases, deployment complexities and more.
:::

With GitHub releases, you can also view and audit the changes between each
release with the code. Another powerful option which is not mandated is to
combine [Semantic Versioning](https://semver.org/) with [Conventional
Commits](https://www.conventionalcommits.org) . This will automatically generate
the release note to detail aspects such as fixes, features, chores etc.

Those those who wish not to use Conventional Commits to generate release notes
another useful feature in GitHub is using labels from the pull request to
generate the release notes. See [here](https://github.com/actions/labeler) and
[here](https://docs.github.com/en/issues/using-labels-and-milestones-to-track-work/managing-labels)
for information on automating labels on the pull request and
[here](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes#configuring-automatically-generated-release-notes)
to use those labels in the automatically generated release note.

:::tip
GitHub has many useful features to automate 🤖 many aspects of your
software delivery giving you more time to focus on the code! Please plan this in
at the beginning of your project. 💪

![image info](assets/cat-computer.gif)
:::
