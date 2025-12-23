# Tokens

When automating in GitHub it may be required to use a Personal Access Token
([PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#about-personal-access-tokens))
for API/CLI access.

If required a GitHub service account should be used, **NOT** your own personal
PAT. Using personal PATs can lead to things breaking when individuals delete
their PATs or leave the company!

To create a GitHub service account please follow the guide in
Confluence in within the Technology Tooling team [here](https://directline.atlassian.net/wiki/spaces/STS/pages/11837669593/Creating+a+GitHub+Automation+Service+Account)

## Types of token

Two types of tokens can be created:

- [fine grained](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#fine-grained-personal-access-tokens)
- [classic](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#personal-access-tokens-classic)

**Fine-grained** tokens are recommended in DL as the name suggests they provide
the capability to apply more granular permissions and must have an expiration
date to support rotation.

It is also not necessary to always use a PAT in workflows as the
built-in `GITHUB_TOKEN` can be used. See this link for more information
[Automatic token authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)
