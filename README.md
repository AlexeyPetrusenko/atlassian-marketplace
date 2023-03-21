# Atlassian Marketplace Release Action

This repo defines composite actions to build and release server app to Atlassian marketplace.

## perform-release
Perform `atlas-mvn release:prepare` with given version.

### Inputs
|Name| Required | Description                                        |
|---|----------|----------------------------------------------------|
| `branch` | Yes      | The release branch to checkout                     |
| `version` | No       | The version to release                             |
| `githubUserName` | Yes      | The GitHub username to use                         |
| `githubUserEmail` | Yes      | The GitHub user email to use                       |
| `azureToken` | Yes      | The Azure token to load libraries from artifactory |
| `skipTests` | No       | Skip tests during operation                        |

### Outputs
|Name| Description      |
|---|------------------|
| `released-tag` | The released tag |

## setup-node
Setup node environment if the `package.json` is available at the root of checked out repository.
<p>This action installs `npm` and `node` using engine information of `package.json`. If `package.json` is available but neither `node` nor `npm` is specified then it will fail.

### Inputs
None

### Outputs
|Name|Description|
|---|---|
| `is-node-project` | `true` if the current repository is node project, `false` otherwise |
| `node-version` | The installed `node` version |
| `npm-version` | The installed `npm` version |

## build-tag
Build server app into jar files using `atlas-mvn clean package` command and upload the result as assets to GitHub release.

### Inputs
| Name          | Required | Description                                                        |
|---------------|----------|--------------------------------------------------------------------|
| `tag`         | Yes      | The tag to build                                                   |
| `skipTests`      | No       | Skip tests during operation                                        |
| `githubToken` | Yes      | The GitHub PTA to upload the jar files as assets to GitHub release |

### Outputs
|Name| Description                                                                                                                                                                |
|---|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `addon-key` | The app addon-key extracted from `atlassian-plugin.xml`                                                                                                                    |
| `should-bump-version` | `true` if the `release-version`:<br /> <ul><li>is different from the one specified in `pom.xml`AND</li><li>is tagged at head of release branch</li></ul>`false` otherwise. |
| `file-path` | The relative path to the jar file                                                                                                                                          |
| `file-name` | The jar file name in format `artifactId`.jar                                                                                                                               |
| `version` | The version name                                                                                                                                                           |

## marketplace
Release the app to Atlassian marketplace and bump the required version for server to cloud migration.
<p>This action makes a new release for an existing app. It will fail if the app does not exist.

### Inputs
| Name               |Required| Description                     |
|--------------------|---|---------------------------------|
| `addonKey`         | Yes | The app addon-key               |
| `filePath`         | Yes | Path to the jar file            |
| `fileName`         | Yes | The name of the jar file        |
| `version`          | Yes | The new version to release      |
| `marketplaceUser`  | Yes | The Atlassian marketplace user  |
| `marketplaceToken` | Yes | The Atlassian marketplace token |
| `releaseSummary`   | No       | Release summary                 |
| `releaseNotes`     | No       | Release notes                   |

### Outputs
None

## get-release-notes
Gets the release notest content between `tag` and previous `tag`. By default is looking for files in `release-notes/*.txt`
Restrictions: only one release notes files can exist between twow tags.

### Inputs
| Name          | Required | Description                                                        |
|---------------|----------|--------------------------------------------------------------------|
| `tag`         | Yes      | The tag to check                                                   |
| `releaseNotesDirectory`      | No       | Directory of release notes                                         |
| `releaseNotesPattern`      | No       | Release notes files pattern, default: *.txt                        |
| `githubToken` | Yes      | The GitHub PTA to upload the jar files as assets to GitHub release |

### Outputs
|Name| Description                                                                                                                                                                |
|---|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `notes` | Notes content if exists                                                                                                                                                    |


## release
Combine 4 actions above to perform full release process:
* perform-release
* build-jar
* get-release-notes
* markeplace


### Inputs
|Name| Required | Description                         |
|---|----------|-------------------------------------|
| `branch` | No       | The release branch to checkout      |
| `version` | No       | The version to release              |
| `githubUserName` | Yes      | Github username to commit           |
| `githubUserEmail` | Yes      | Github user email to commit         |
| `githubToken` | Yes      | The GitHub PTA to checkout the repo |
| `marketplaceUser` | Yes      | The Atlassian marketplace user      |
| `marketplaceToken` | Yes      | The Atlassian marketplace token     |
| `azureToken` | Yes      | The Azure token to load libraries   |
| `releaseNotesDirectory` | No       | Release notes directory             |
| `releaseNotesPattern`      | No       | Release notes files pattern, default: *.txt  |
| `releaseSummary` | No       | Release summary                     |
| `skipTests` | No       | Skip tests during operations        |

### Outputs
None


### Sample

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to release'
        required: false
        default: ''
      releaseSummary:
        description: 'The release summary'
        required: false  
      skipTests:
        description: 'Skip tests'
        type: boolean
        required: false

jobs:
  release:
    runs-on: ubuntu-latest
    name: Perform release cycle
    steps:
      - name: Release
        uses: AlexeyPetrusenko/atlassian-marketplace/release@master
        with:
          branch: ${{ github.ref_name }}
          version: ${{ inputs.version }}
          releaseSummary: ${{ inputs.releaseSummary }}
          githubUserName: ${{ secrets.GITHUB_USER_NAME }}
          githubUserEmail: ${{ secrets.GITHUB_USER_EMAIL }}
          githubToken: ${{ secrets.GITHUB_TOKEN }}
          marketplaceUser: ${{ secrets.MARKETPLACE_USER }}
          marketplaceToken: ${{ secrets.MARKETPLACE_TOKEN }}
          azureToken: ${{ secrets.AZURE_TOKEN }}
          releaseNotesDirectory: 'release-notes'
          releaseNotesPattern: '*.txt'
          skipTests: ${{ inputs.skipTests }}
```