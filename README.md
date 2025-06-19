# Examples of configuring Github Actions to use the OIDC integration with JFrog

The desired outcome of these example workflows is to enforce a process and not allow teams to bypass it. The ideal setup would be a workflow_dispatch to a reusable workflow that defines a secret, which only the workflow_call repo would be able to see. However, this is not possible. Secrets and variables have to be defined by the calling workflow and declared as part of the workflow_dispatch trigger.

## Example 1 - Limit by workflow name and/or repository

The setup for this example is the standard OIDC integration from the [JFrog OIDC example repo](https://github.com/jfrog/jfrog-github-oidc-example). The enforcement is configured in the Identity Mapping within the JFrog platform. Adding the following JSON claim:
```
{"repository":"dtlaycock/jfrog-github-oidc-example"}
```

Or any combination of similar restrictions (the examples overlap and are from least to most restrictive):
```
{
    "workflow": "Build & Publish",
    "repository": "dtlaycock/jfrog-github-oidc-example",
    "workflow_ref": "dtlaycock/jfrog-github-oidc-example/.github/workflows/build-publish.yml@refs/head/main"
}
```

That means that the token exchange would only work for this github repository. The configuration values for the JFrog CLI could be implemented as secrets or variables at the Organization level. That would make them discoverable but they couldn't be used in workflows in other repositories due to the limitations imposed by the Identity Mapping.

See [build & publish workflow](.github/workflows/build-publish.yml)

## Example 2 - Bypass the secrets restriction by using an intermediary service

This example doesn't have any restrictions in the identity mapping on the JFrog side. The centralized workflow has secrets defined and it expects to be triggered via a call to the Github Actions dispatch API, which would include a JSON payload to indicate which repository should be built, the artifactory repository to use when resolving the dependencies during the build process and the artifactory repository to publish the build artifact(s) to.

The cURL command to trigger the workflow would look like this:
```
curl --silent --location 'https://api.github.com/repos/dtlaycock/jfrog-github-oidc-example/dispatches' \
--header 'Accept: application/vnd.github+json' \
--header 'X-GitHub-Api-Version: 2022-11-28' \
--header 'Content-Type: application/json' \
--header 'Authorization: ••••••' \
--data '{
    "event_type": "build-publish",
    "client_payload": {
        "github_repository": "dtlaycock/test-team-repo",
        "resolution_repo": "dominicl-npm-virtual",
        "publish_repo": "dominicl-npm-virtual"
    }
}'
```

See [Repository Dispatch Workflow](.github/workflows/repository-dispatch.yml)
