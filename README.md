# How to publish a module to TFE without VCS  
  
## **Listing and reading modules, providers and versions**  
  
* List all modules  
    * ```  
        curl -H "Authorization: Bearer $TOKEN" \  
            https://app.terraform.io/api/registry/v1/modules/pphan/ \  
            | jq  
    * Replace "**pphan**" with your org name  
  
## Create a Module  
  
* `POST /registry-modules/:organization_name/:name/:provider/versions`  
* Creates a new registry module without a backing VCS repository.  
    * After creating a module, a version must be created and uploaded in order to be usable.  
    * Modules created this way do not automatically update with new versions.  
    * Instead, you must explicitly create and upload each new version with the [<u>Create a Module Version][1]</u> endpoint.  
* Create "**createmodule.json**"  
    * ```  
        {  
          "data": {  
            "type": "registry-modules",  
            "attributes": {  
              "name": "my-module",  
              "provider": "aws"  
            }  
          }  
        }  
  
* Sample Request  
    * ```  
        curl \  
          --header "Authorization: Bearer $TOKEN" \  
          --header "Content-Type: application/vnd.api+json" \  
          --request POST \  
          --data @createmodule.json \  
          https://app.terraform.io/api/v2/organizations/pphan/registry-modules | jq  
  
    * Replace "**pphan**" with your org name  
  
## **Create a Module Version**  
  
* POST /registry-modules/:organization_name/:name/:provider/versions  
* POST /registry-modules/:phan/:my-module/:aws/versions  
* Creates a new registry module version.  
    * This endpoint only applies to modules without a VCS repository; VCS-linked modules automatically create new versions for new tags.  
    * After creating the version, the module should be uploaded to the returned upload link.  
* Create "createmoduleversion.json"  
    * ```  
        {  
          "data": {  
            "type": "registry-module-versions",  
            "attributes": {  
              "version": "1.0.0"  
            }  
          }  
        }  
  
* Sample Request  
    * ```  
        curl \  
          --header "Authorization: Bearer $TOKEN" \  
          --header "Content-Type: application/vnd.api+json" \  
          --request POST \  
          --data @createmoduleversion.json \  
          https://app.terraform.io/api/v2/registry-modules/pphan/my-module/aws/versions | jq  
  
    * Replace "**pphan**" with your org name  
    * Replace "**my-module**" with your module name.  
* Sample Response  
    * ```  
        {  
          "data": {  
            "id": "modver-qjjF7ArLXJSWU3WU",  
            "type": "registry-module-versions",  
            "attributes": {  
              "source": "tfe-api",  
              "status": "pending",  
              "version": "1.2.3",  
              "created-at": "2018-09-24T20:47:20.931Z",  
              "updated-at": "2018-09-24T20:47:20.931Z"  
            },  
            "relationships": {  
              "registry-module": {  
                "data": {  
                  "id": "1881",  
                  "type": "registry-modules"  
                }  
              }  
            },  
            "links": {  
              "upload": "https://archivist.terraform.io/v1/object/dmF1bHQ6djE6NWJPbHQ4QjV4R1ox..."   <---  
            }  
          }  
        }  
  
    * Copy/note the **.links.upload** URL. You will need this **Unique Object ID** for the next step.  
  
## **Upload a Module Version**  
  
* `PUT https://archivist.terraform.io/v1/object/<UNIQUE OBJECT ID>`  
* **The URL is provided in the upload links attribute in the registry-module-versions resource**.  
* **Expected Archive Format**  
    * Terraform Cloud expects the module version uploaded to be a tarball with the module in the root (not in a subdirectory).  
    * Given the following folder structure:  
        * ```  
            terraform-null-test  
            ├── README.md  
            ├── examples  
            │   └── default  
            │       ├── README.md  
            │       └── main.tf  
            └── main.tf  
  
    * Package the files in an archive format by running tar zcvf module.tar.gz * in the module's directory.  
    * Here is a sample where I cloned a repo and then packaged it up to be sent out.  
        * ```  
            git clone https://github.com/hashicorp/terraform-aws-consul.git  
            cd terraform-aws-consul/  
            tar zcvf module.tar.gz *  
  
* **Sample Request**  
    * ```  
        curl \  
          --header "Content-Type: application/octet-stream" \  
          --request PUT \  
          --data-binary @module.tar.gz \  
        https://archivist.terraform.io/v1/object/dmF1bHQ6djE6TTl5RmRjVU9xd1NLMHV5aWIzVFpnQ1FRbjRQQnA4K2hNZFNRa2psNXJrVGNTc0hoVmFWOVJZcnlmK0dNVFhoeUt0SHM2VnkvQnpmcDFjUWVpNm5pQk9BTjRUS1g4OUZENGtOQ055UW56SisvZXBTbXNGTEpIckZ6OERQVU1aYUxzbWR0L2Nkbjk1VGZGZmhIaW5DaGRRVWJ3MGljanVYSGdOOVhMcml4MkoxeFFFdE1xeGp4MnlDZkpEcmlhSE5pbGV0Q2NuRS8wYWlyMEZ5em5NNlVqbWxDNUoxcTJLRDZWMS9pMk0zSGFKU1FZR0pyN3dEZ1l2bzlqUUZpdGxvMS9YSlhmSk51TGljUVdDWEFnVXppb09pL0tNVjlZWVJPV092aUExVzkyUT09  
  
    * **NOTE**: Object ID URL came from module version response.  
Source: https://www.terraform.io/docs/cloud/api/modules.html

* List all modules  
    * ```  
        curl -H "Authorization: Bearer $TOKEN" \  
            https://app.terraform.io/api/registry/v1/modules/pphan/ \  
            | jq  
  
    * Replace "**pphan**" with your org name  
    * Expected output  
        * ```  
            ...  
                {  
                  "id": "pphan/my-module/aws/1.2.4",  
                  "owner": "",  
                  "namespace": "pphan",  
                  "name": "my-module",  
                  "version": "1.2.4",  
                  "provider": "aws",  
                  "description": "",  
                  "source": "",  
                  "tag": "",  
                  "published_at": "2019-09-26T00:00:19.536592Z",  
                  "downloads": 0,  
                  "verified": false  
                }  
            ...  
  
## Delete a Module  
  
* ```  
    curl \  
      --header "Authorization: Bearer $TOKEN" \  
      --header "Content-Type: application/vnd.api+json" \  
      --request POST \  
      https://app.terraform.io/api/v2/registry-modules/actions/delete/pphan/my-module  
  
* Replace "**pphan**" with your org name  
* Replace "my-module" with your module name.  
  
[1]: https://www.terraform.io/docs/cloud/api/modules.html#create-a-module-version  
