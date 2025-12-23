# Best Practices

## Project Configuration

- Create project specific remote repositories (E.g.: dap-pypi), rather than
  publicly available pypi. Recommended to store all remote repositories within a
  single Virtual repository (all-remote).
  - This is to ensure we can create ignore rules for a projects remote/proxied
    resources where required.
- Use DEV & PROD repositories to separate build artifact's. E.g.:
  dap-pypi-dev/prod. Upload final release Artifact's to PROD repositories
  - No more than the 3 latest Builds should be stored in DEV.
  - Simplifies which resources can be deleted (DEV) to save on storage.
  - Storage of PROD builds should be agreed with your CISO representative and
    audit requirements.
  - Allows re-indexing of PROD artifact's to maintain Xray scanning.
- Do not store historic resources in Local repositories within Artifactory (an
  artifact is considered historic if it will be stored for longer than 90 days).
  - Xray will not update Vulnerabilities on artifact's older than 90 days.
  - Historic artifact's can be added as remote resources. This will ensure Xray
    is always enabled.
- Use separate workflows for build and deploy.
  - Build: Xray currently blocks Vulnerable packages from download, resolve
    potential Vulnerabilities before deploying artifact.
  - Deploy: re-deploy older builds after Xray has dropped artifact from its
    index. Xray will scan artifact's on upload/deployment to Artifactory.
- Projects should be kept under `10GB` (Project Admins responsibility).
- Build flows should not be run excessively. Excessive use will be billed to teams
  - Build flows should be used to determine Vulnerabilities. Once a build
    artifact is made it can be uploaded to Artifactory for more detailed monitoring.
- For multiple teams within a project. Ensure each repository is clearly named
  to a team or service.

## Xray Scanning

- Xray scanning only occurs on upload of new artifact's. Xray data is deleted
  after 90 days and will no longer receive security updates.
  - Use Xray “Scan Lists“ to see current Violations on build artifact's.
    Alternatively you can see Xray data on individual components/Artifacts
  - Reupload Production Artifacts semi-regularly to ensure Xray is enabled.
