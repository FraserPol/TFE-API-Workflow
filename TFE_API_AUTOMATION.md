## Pre-reqs
- Org is created
- User is an admin

First thing we need to do is get an org token, we can do that via the GUI interface or the API - https://www.terraform.io/docs/enterprise/api/organization-tokens.html

To do this via the GUI, simply log in as an admin to TFE and select your organization in the top left
-> Organization Settings
-> API token
In the bottom under Organization settings select Regenerate

```
export TOKEN="your_org_token_here"
Example:
export TOKEN="AxqiATnUHR5u6A.atlasv1.JTRapSQ7qrqpNyBqR8bjlzDeaLKmYiIKJUIXAIfcz6ewdNzXILuUTs0YumJ8Izuaaaa"
```

## Workspace Creation
###GUI
You can create a workspace without VCS by selecting "Source 'None'" in the GUI:
```
https://MY_TFE_DOMAIN/app/MY_TFE_ORGANIZATION/workspaces/new
```


###API
The first step is to create a Workspace without the VSC backing - https://www.terraform.io/docs/enterprise/api/workspaces.html#create-a-workspace

The workspace_payload.json exmaple is in this repo. The POST is below;
```
curl --header "Authorization: Bearer $TOKEN" \
  --header "Content-Type: application/vnd.api+json" \
  --request POST \
  --data @workspace_payload.json \
  https://MY_TFE_DOMAIN/api/v2/organizations/MY_TFE_ORGANIZATION/workspaces | jq
```

The response will be as such;
```
{
  "data": {
    "id": "ws-DZFWKcGK8QYsuChb",
    "type": "workspaces",
    "attributes": {
      "name": "company-demo",
      "environment": "default",
      "auto-apply": false,
      "locked": false,
      "created-at": "2018-07-10T16:39:20.269Z",
      "working-directory": null,
      "terraform-version": "0.11.7",
      "vcs-repo": null,
      "permissions": {
        "can-update": true,
        "can-destroy": true,
        "can-queue-destroy": true,
        "can-queue-run": true,
        "can-update-variable": true,
        "can-lock": true,
        "can-read-settings": true
      },
      "actions": {
        "is-destroyable": true
      }
    },
    "relationships": {
      "organization": {
        "data": {
          "id": "GCP",
          "type": "organizations"
        }
      },
      "latest-run": {
        "data": null
      },
      "current-run": {
        "data": null
      }
    },
    "links": {
      "self": "/api/v2/organizations/GCP/workspaces/company-demo"
    }
  }
}
```


## Variables

post some variables for our workspace - https://www.terraform.io/docs/enterprise/api/variables.html

You can also use the TFE-CLI

###GUI:

Easier to get started with. Just go to your workspace in the GUI. You can also use the TFE-CLI tool.
```
https://MY_TFE_DOMAIN/app/MY_TFE_ORGANIZATION/company-demo/variables
```
###API:
To do this in an efficient manner you can loop through - https://gist.github.com/FraserPol/d6eedef08eb47a0f090b510f5029fe84
Pull the workspace ID from the workspace creation command above. That workspace ID goes in the variable_payload.json file.
```

curl -k \
  --header "Authorization: Bearer $TOKEN" \
  --header "Content-Type: application/vnd.api+json" --data @variable_payload.json \
"https://MY_TFE_DOMAIN/api/v2/vars?filter%5Borganization%5D%5Bname%5D=MY_TFE_ORG&filter%5Bworkspace%5D%5Bname%5D=MY_TFE_WORKSPACE"


curl --header "Authorization: Bearer $TOKEN" \
  --header "Content-Type: application/vnd.api+json" \
  --request POST \
  --data @variable_payload.json \
  https://personatwork.com/api/v2/vars | jq
```
The response will be as such;
```
{
  "data": {
    "id": "var-jHvEWbaSoJi3Vmva",
    "type": "vars",
    "attributes": {
      "key": "boo",
      "value": "baz",
      "sensitive": false,
      "category": "terraform",
      "hcl": false
    },
    "relationships": {
      "configurable": {
        "data": {
          "id": "ws-DZFWKcGK8QYsuCaa",
          "type": "workspaces"
        },
        "links": {
          "related": "/api/v2/organizations/GCP/workspaces/company-demo"
        }
      }
    },
    "links": {
      "self": "/api/v2/vars/var-jHvEWbaSoJi3Vmva"
    }
  }
}
```
## Uploading Config
Our workspace and variables are set. We need to start trigger a run - https://www.terraform.io/docs/enterprise/api/run.html

Note: You can also use the TFE-CLI tool for these steps.

### Steps 1 - Create a configuration -

https://www.terraform.io/docs/enterprise/api/configuration-versions.html#create-a-configuration-version

Note: this requires a user token, a user token can be created via tha API or;

-> Organization Settings
-> API token
-> User Tokens
-> Generate Token

```
export TOKEN="your_org_token_here"
```

Use the workspace ID from above in the url
```
curl  --header "Authorization: Bearer $TOKEN" --header "Content-Type: application/vnd.api+json" --request POST --data @config_create.json https://MY_TFE_DOMAIN/api/v2/workspaces/ws-DZFWKcGK8QYsuChb/configuration-versions | jq
```

The response should be similair;
```
{
  "data": {
    "id": "cv-FWDhWq1EGsvtMLiq",
    "type": "configuration-versions",
    "attributes": {
      "auto-queue-runs": true,
      "error": null,
      "error-message": null,
      "source": "tfe-api",
      "status": "pending",
      "status-timestamps": {},
      "upload-url": "https://person.com/_archivist/v1/object/dmF1bHQ6djE6Lzl3WFZKMEw1Y1lRUk52SzdWUmZFRnZacDUzVFQrYmxwNVhiS3BLdnB3UDdOSkdyYXRYYWNodEx3RTJNNllWa3RuVkNHQW5HWlJDcEU5clhBamZOT0lpdmwydFJJYzhra0g4TnNQQjcxdnNUS1RGRGFLcFlzdmlJdUNsRndpTWNPR3hnb1hJYWFTRXBkNnV5a1dMSDZzY3BnS1R1aFMzbisyRG5udi9TbGplZkljcjYrbk1ReitYOEVlc2RCb1dqcTBEWEN1YjZTeTI1Q0h2SEdIOUs2MStraWREVlJFd2w4SERidloxQlBuN0ZrbVBrSFJlQ1B2d0Q0UTdDbkJIYmVwS1BrRE9DbG9wSUVzbEhJVFVTRXRtRDVVNkI5TXM9"
    },
    "relationships": {
      "ingress-attributes": {
        "data": null,
        "links": {
          "related": "/api/v2/configuration-versions/cv-FWDhWq1EGsvtMLiq/ingress-attributes"
        }
      }
    },
    "links": {
      "self": "/api/v2/configuration-versions/cv-FWDhWq1EGsvtMLiq"
    }
  }
}
```

use the "upload-url" for the next command to upload the terraform code.

## Step 2 - Upload a configuration -

https://www.terraform.io/docs/enterprise/api/configuration-versions.html#upload-configuration-files

A configuration is a snapshot in time of the configuration from your VSC Repo. The files need to be compressed using tar

```
tar -C azure-vault-mysql-demo/ -czf azure-mysql.tar.gz .
```

```
curl -k  \
  --header "Content-Type: application/octet-stream" \
  --request PUT \
  --data-binary @azure-mysql.tar.gz \
  https://MY_TFE_DOMAIN/_archivist/v1/object/dmF1bHQ6djE6Lzl3WFZKMEw1Y1lRUk52SzdWUmZFRnZacDUzVFQrYmxwNVhiS3BLdnB3UDdOSkdyYXRYYWNodEx3RTJNNllWa3RuVkNHQW5HWlJDcEU5clhBamZOT0lpdmwydFJJYzhra0g4TnNQQjcxdnNUS1RGRGFLcFlzdmlJdUNsRndpTWNPR3hnb1hJYWFTRXBkNnV5a1dMSDZzY3BnS1R1aFMzbisyRG5udi9TbGplZkljcjYrbk1ReitYOEVlc2RCb1dqcTBEWEN1YjZTeTI1Q0h2SEdIOUs2MStraWREVlJFd2w4SERidloxQlBuN0ZrbVBrSFJlQ1B2d0Q0UTdDbkJIYmVwS1BrRE9DbG9wSUVzbEhJVFVTRXRtRDVVNkI5TXM9
```


## Step 3 - Runs
A run should have been kicked off after we uploaded our tarball. You can also manually trigger runs using the following:

https://www.terraform.io/docs/enterprise/api/run.html#create-a-run

Just substitute the configuration version id and the workspace id.

Note: You can also trigger runs with the TFE-CLI in a simple manner.
