# Advanced Security

JFrog's Advanced Security capabilities provides DL engineers:

| Feature                                    | Description                                                                                |
| ------------------------------------------ | ------------------------------------------------------------------------------------------ |
| Secret Detection                           | Finds hardcoded secrets (e.g., API keys, passwords) in source code and binaries            |
| IaC Security                               | Scans Infrastructure as Code files (e.g. Terraform, Kubernetes YAML) for misconfigurations |
| Software Composition Analysis (SCA)        | Detect vulnerabilities from public and premium vulnerability databases                     |
| Static Application Security Testing (SAST) | Detect vulnerabilities and prevent code security breaches with analysis on our source code |
| Malicious Packages Detection               | Detect packages from the SBOM that are known to be malicious                               |

To utilise these capabilities and integrate tools to ensure proactive security
risk identification (shift left) and mitigation the following are available.

## IDE Extensions/Plugins

Scanning in the IDE for the developer allows for earlier detection and mitigation
of the vulnerabilities. Earlier mitigation of vulnerabilities reduces the effort
and time take to fix issues vs detecting them in built systems and products during
the build process.

Plugins are available for several IDEs. See below for the guides for use and
installation.

- [Visual Studio Code](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/ides/visual-studio-code/installation)
- [JetBrains](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/ides/jetbrains/installation)
- [Eclipse](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/ides/eclipse/installation)
- [Visual Studio](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/ides/visual-studio/installation)

:::tip
We recommend using Single Sign-On (SSO) to authenticate to the DL JFrog instance
where possible.
:::

:::warning
The Visual Studio plugin does not support all Advanced Security capabilities!
:::

The results of the scan are shown with the Plugin, file editor and JFrog console
under `Xray -> On-Demand Scanning -> Source code scans`.

Plugin

![jas-ide png](./assets/jas-ide.png)

File Editor

![jas-file-editor png](./assets/jas-file-editor.png)

JFrog Console

![jas-jf-console png](./assets/jas-jf-console.png)

### Post Installation

![warning gif](./assets/warning.gif)

The default installation of the IDE plugin has a number of paths excluded:

`.git,test,venv,node_modules`

We recommend the default exclusions are updated because:

1. If your root directory (typically your repository name) contains the word
   `test` it results in all files in the repository being ignored by the secret scanner.
1. If your root directory contains any sub directories containing the word
   `test` then anything within those directories being ignored by the secret scanner.

When we contacted JFrog this is apparently by design...

![confused gif](./assets/confused.gif)

To update the exclusion list open the IDE plugin settings and replace the
exclusions glob pattern with:

`**/*{.git,venv,node_modules,target}*`

You can also add more if required by your package manager.

To further customise your extension/plugin see the `Manage` section of each in
the JFrog documentation. E.g. Visual Studio Code [Manage](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/ides/visual-studio-code/manage)
.

## Frogbot

[Frogbot](https://github.com/jfrog/frogbot) is a Git-integrated bot that helps
us shift security left by identifying and fixing secrets/vulnerabilities early
in the software development lifecycle directly within pull requests and commits.

### GitHub

Frogbot provide a [GitHub action](https://github.com/jfrog/frogbot/) to use in
workflows.

Below is an example of a workflow which runs on several events:

- PR (opened and synchronised)
- Daily (cron)
- On demand

```YAML
name: "Frogbot Security Scan"

on:
  pull_request_target:
    types: [opened, synchronize]
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

permissions:
  pull-requests: write
  contents: write
  security-events: write
  id-token: write

jobs:
  frogbot-scan:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        branch: ["main"] # Add further branches if required
    steps:
      - uses: jfrog/frogbot@v2
        with:
          oidc-provider-name: ${{ vars.JFROG_OIDC_PROVIDER_NAME }}
          oidc-audience: ${{ vars.JFROG_OIDC_AUDIENCE }}
        env:
          JF_PROJECT: project
          JF_URL: ${{ vars.JFROG_SERVER_URL }}
          JF_GIT_TOKEN: ${{ secrets.JF_GITHUB_TOKEN }}
          JF_GIT_BASE_BRANCH: ${{ matrix.branch }}
          JF_PR_SHOW_SECRETS_COMMENTS: true
```

:::note
This workflow uses the recommended authentication method OIDC. See the JFrog [OIDC](../../security/oidc/04-jfrog.md)
section for more details.

The role this workflow assumes through the identity mapping requires the
`Manage Xray Data` permission to complete the scan and upload the results.
:::

On a successful scan in the PR event, Frogbot will decorate the PR with comments
when issues are found:

![result1 png](./assets/frogbot-result1.png)

![result2 png](./assets/frogbot-result2.png)

Or if nothing was found:

![good png](./assets/frogbot-good.png)

With `security-events: write` it provides Frogbot permission to add issues to
the Security tab `Code scanning` section of the GitHub repository.

:::note
At the time of this guide `Advanced Security` [Code scanning](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning)
is an additional cost at the Organisational level and disabled even when JFrog
populates it. Options are being worked on to allow the section without additional
costs.
:::

![result3 png](./assets/frogbot-result3.png)

The full Frogbot [installation](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/frogbot/installation/github-actions/installation)
and [configuration](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/frogbot/configure-frogbot)
guide contains more details on use.

### Other Tools

If you don't use GitHub actions, its still possible to utilise Frogbot but
requires further configuration to run.

Below is a simple example:

```sh
JF_URL="JF_SERVER_URL"
getFrogbotScriptPath=$(if [ -z "$JF_RELEASES_REPO" ]; then echo "https://releases.jfrog.io"; else echo "${JF_URL}/artifactory/${JF_RELEASES_REPO}"; fi)
curl -fLg "$getFrogbotScriptPath/artifactory/frogbot/v2/[RELEASE]/getFrogbot.sh" | sh
./frogbot cfpr
```

This script will download the Frogbot executable and run the tool.

Further supported methods can be found in the JFrog [documentation](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/frogbot/installation)
.

:::note
Remember to set the various environment variables for authentication and
configuration of the tool from this [guide](https://jfrog.com/help/r/jfrog-security-user-guide/shift-left-on-security/frogbot/configure-frogbot/frogbot-optional-configuration-parameters)
.
:::

## JFrog CLI

If JFrog doesn't provide a extension/plugin for your IDE, or Frogbot for chosen
CI/CD ecosystem it is still possible to use the JFrog CLI to provide the same
capabilities within a terminal.

First [install](https://jfrog.com/getcli/) the CLI using your chosen method.

Next authenticate to the DL instance.

```text
jf config add
```

Now enter the following when prompted:

```text
Enter a unique server identifier: https://directline.jfrog.io
JFrog Platform URL: https://directline.jfrog.io
Use the arrow keys to navigate: ↓ ↑ → ←
Select one of the following authentication methods:
     Access Token
  🐸 Web Login <--- this one
     Username and Password / Reference token
     Mutual TLS
     OIDC
```

After selecting `Web Login` a window will launch requesting a code to be entered.

```text
[🔵Info] After logging in via your web browser, please enter the code if prompted: 91ee
[🔵Info] Please open the following URL in your browser to authenticate:
[🔵Info] https://directline.jfrog.io/ui/login?jfClientSession=e2ca450b-023c-477e-ba97-87c8fdb261ee&jfClientName=JFrog-CLI&jfClientCode=1
```

Once complete a final message will be shown in the terminal:

```text
You're now logged in!
```

This has authenticated the CLI to JFrog using SSO then a new identity token.

To confirm run `jf rt ping`. The result should be `OK`.

### Scanning

Scanning your source code is by the `jf audit` command.

:::note
`jf audit` will build your software package in order to complete correct
SCA. Please ensure your environment has the necessary environment variables,
paths etc to enable this.
:::

From the root of your repository, run the command. Below is an example of the
output when issues are detected.

![vulnerable png](./assets/vulnerable-dependencies.png)

If you would like to see the results in the JFrog UI for further analysis, visit
the [On-Demand Scanning](https://directline.jfrog.io/ui/onDemandScanning/list?scanType=source-code-scans)
section in the UI.

### Post Installation

![warning gif](./assets/warning.gif)

The JFrog CLI has the same default behaviour as the [IDE extension/plugin](#post-installation)
ignoring paths containing `test` 🫣.

We can ensure paths with `test` are scanned by overriding exclusions using the `--exclusions`
flag:

`jf audit --exclusions "*.git*;*node_modules*;*target*;*venv*;dist"`

For more information on the flag see the [documentation](https://jfrog.com/help/r/jfrog-security-user-guide/developers/cli/scan-your-source-code).
