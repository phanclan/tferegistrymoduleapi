# Publish a module to TFE PMR without VCS  
  
## Source  
  
* [terraform.io/docs/cloud/api/modules.html][1]  
* Roger Berlind  
    * **AddModulestoPMR.md**  
  
## Overview  
  
* The following procedure can be used to add Terraform modules to TFE's [<u>Private Module Registry][2]</u> when they are in a VCS system that is not supported by TFE. Note that we are using the [<u>Registry Modules][3]</u> endpoint of the TFE API.  
* The procedure for adding a module without a backing VCS registry has 5 main steps:  
    * Clone the repository containing the module.  
    * Create the module.  
    * Create a module version.  
    * Package the module into a compressed tar (tar.gz) file.  
    * Upload the tar to the module version's upload URL.  
* **NOTE**: The modules are not automatically updated when changes are made to the source in the VCS repository. If you change the code of the module, you will have to upload a new version of the module by repeating steps 3-5 after doing a git pull command against the repository to pull the changes.  
  
## **Prerequisites**  
  
* Make sure you have a user ID and password for accessing the TFE server you are using.  
* Make sure you have a user API token for the owners team of the organization on the TFE server into which you want to import the module.  
    * You can create this by clicking on your person icon in the upper right corner of the TFE UI and select "**User Settings**".  
    * Then "**Tokens**.  
    * Provide a Description: ex"CLI"  
    * Click the "**Generate token**" button.  
    * **TIP**: Save your token in a secure location since the TFE UI will not display it again.  
  
## **Step 1: Clone the Repository with the Module**  
  
* Clone the repository of the module to your laptop if you do not already have it. If you do have it, you should run `git pull` to make sure it is up-to-date.  
* Use a command like git clone <repository>. You might have to provide a Git username and password. Note that the repository will be cloned into a directory under your current directory.  
    * `git clone https://github.com/hashicorp/terraform-aws-consul.git`  
* Then cd into the directory containing the cloned repository.  
  
## **Step 2: Create a Module**  
  
* In this step, you will use the [<u>Create a Module][4]</u> TFE API to create a module that does not have a backing VCS repository.  
* Export your user API token for the TFE server  
    * `export TFE_TOKEN=<your_token>`  
* Create a file called `<module_name>_module.json` where "`<module_name>`" is the name of your module. ex `consul_module.json`  
* Copy the JSON text from [<u>create module sample payload][5]</u> into it.  
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
  
* Change the module's name, "**my-module**", to "<module_name>".  
    * ```  
        {  
          "data": {  
            "type": "registry-modules",  
            "attributes": {  
              "name": "consul",  
              "provider": "aws"  
            }  
          }  
        }  
  
* Set the provider to "**aws**", "**azure**", "**google**" or the other primary provider used to provision resources in the module.  
* Save the file.  
* Run a curl command like the following to create the module. Replace <module_name> with the name of your module, use your TFE organization name instead of <organization> and use the actual DNS name of your TFE server instead of <tfe_server>.  
    * ```  
        curl \  
          --header "Authorization: Bearer $TOKEN" \  
          --header "Content-Type: application/vnd.api+json" \  
          --request POST \  
          --data @consul_module.json \  
          https://app.terraform.io/api/v2/organizations/pphan/registry-modules | jq  
  
    * Replace "**consul_module.json**" with your own.  
    * Replace "**app.terraform.io**" with your TFE URL.  
    * Replace "**pphan**" with your org name.  
* If this is successful, you will get back JSON like this:  
    * ```  
        {"data":{"id":"mod-SbEfgQpAcNpB3ZM2","type":"registry-modules","attributes":{"name":"<module_name>","provider":"<provider>","status":"pending","version-statuses":[],"created-at":"2019-05-23T17:57:03.971Z","updated-at":"2019-05-23T17:57:03.971Z","permissions":{"can-delete":true,"can-resync":true,"can-retry":true}},"relationships":{"organization":{"data":{"id":"org-w9keXYdKaEuYwXHc","type":"organizations"}}},"links":{"self":"/api/v2/registry-modules/show/<organization>/<module_name>/<provider>"}}}  
  
* If you get something like `{"errors":[{"status":"400","title":"JSON body is invalid","detail":"765: unexpected token at '\u001b'"}]}`, check that your JSON is valid.  
* If you get JSON with a 403 status, make sure you exported your TFE_TOKEN correctly and that you are a member of the owners team in the organization you used in the curl command.  
* NOTE:  
    * After creating a module, a version must be created and uploaded in order to be usable.  
    * Modules created this way do not automatically update with new versions.  
    * Instead, you must explicitly create and upload each new version with the [<u>Create a Module Version][6]</u> endpoint.  
  
## **Step 3: Create a Module Version**  
  
* In this step, you will use the [<u>Create a Module Version][7]</u> TFE API to create a module version for the module you created in Step 2.  
* Create a file called <module_name>_version.json where "<module_name>" is the name of your module.  
* Create "**consul_version.json**"  
    * Copy the JSON text from [<u>create module version sample payload][8]</u> into it.  
    * ```  
        {  
          "data": {  
            "type": "registry-module-versions",  
            "attributes": {  
              "version": "1.0.0"  
            }  
          }  
        }  
  
    * Change the version to the version number you want to use such as "**1.0.0**".  
* Run a curl command to create the module version.  
    * ```  
        curl \  
          --header "Authorization: Bearer $TOKEN" \  
          --header "Content-Type: application/vnd.api+json" \  
          --request POST \  
          --data @consul_version.json \  
          https://app.terraform.io/api/v2/registry-modules/pphan/my-module/aws/versions | jq  
  
    * Generic Sample  
        * ```  
            curl --header "Authorization: Bearer $TFE_TOKEN" \  
                 --header "Content-Type: application/vnd.api+json" \  
                 --data @<module_name>_version.json https://<tfe_server>/api/v2/registry-modules/<organization>/<module_name>/<provider>/versions  
  
    * **NOTE**: Make the obvious substitutions.  
* If this is successful, you will get back JSON like this:  
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
  
    * ```  
        {"data":{"id":"modver-bTMyAgmn6tQoL4dU","type":"registry-module-versions","attributes":{"source":"tfe-api","status":"pending","version":"1.0.0","created-at":"2019-05-23T18:01:15.908Z","updated-at":"2019-05-23T18:01:15.908Z"},"relationships":{"registry-module":{"data":{"id":"1","type":"registry-modules"}}},"links":{"upload":"https://<ptfe_server>/_archivist/v1/object/dmF1bHQ6djE6RmJENGFoVEVmT3plQ095K0h2RUtSejEwWnNpWnEyN2E2YXA1VlZSeCtFbWo4dkp0UzhZTnRjeEtFYVpGTElhQUpkZVoxRXpvSlI4SkhmMFNseWJEZkhrQndrM0IyR3ZxeC9abklzYlROVTZXcWNKQ2Rkb1Bod0hUSGtIaVJPRkhoVEk1YVBVempIdUJGS0svSWVhNysvNk9RZW1FMmdtcUpISW82MHlXd2ZFeGY4cHRQdzgvMTdmU2k4eGI0emg4QVR5Y3ltNVZOamg5dHZwR2dpSXE3Yi9jYW41RnJMV0REMTFKKzhjSFgvZ3FjbHUvQS9EeERpZTdxRklTZjBWZEI4SExrRXhYNGwzTXF6bHZwWk1jSnhtT0ZudE9DM3B5TzFiYWpnK0tPdz09"}}}  
  
* **TIP**: Copy/note the **.links.upload** URL. You will need this **Unique Object ID** upload URL for the next step.  
* Note the upload URL which you will use in Step 5.  
  
## **Step 4: Package Your Module**  
  
* Terraform Cloud expects the module version uploaded to be a tarball with the module in the root (not in a subdirectory).  
* In this step, you will package your module into a compressed tar file.  
* Go into the directory containing the module you are uploading.  
    * `cd terraform-aws-consul`  
* Tar the contents of the module's directory  
    * `tar zcvf <module_name>_module.tar.gz *`  
* **TIP**: Don't run the tar command from the directory above the one containing your module since we want the module's files in the root directory of the tar file.  
  
## **Step 5: Upload the Packaged Module**  
  
* In this step, you will upload the packaged module.  
* Run a command like the following.  
    * ```  
        curl \  
          --header "Content-Type: application/octet-stream" \  
          --request PUT \  
          --data-binary @<module_name>_module.tar.gz \  
        https://archivist.terraform.io/v1/object/dmF1bHQ6djE6TTl5RmRjVU9xd1NLMHV5aWIzVFpnQ1FRbjRQQnA4K2hNZFNRa2psNXJrVGNTc0hoVmFWOVJZcnlmK0dNVFhoeUt0SHM2VnkvQnpmcDFjUWVpNm5pQk9BTjRUS1g4OUZENGtOQ055UW56SisvZXBTbXNGTEpIckZ6OERQVU1aYUxzbWR0L2Nkbjk1VGZGZmhIaW5DaGRRVWJ3MGljanVYSGdOOVhMcml4MkoxeFFFdE1xeGp4MnlDZkpEcmlhSE5pbGV0Q2NuRS8wYWlyMEZ5em5NNlVqbWxDNUoxcTJLRDZWMS9pMk0zSGFKU1FZR0pyN3dEZ1l2bzlqUUZpdGxvMS9YSlhmSk51TGljUVdDWEFnVXppb09pL0tNVjlZWVJPV092aUExVzkyUT09  
  
    * Replace <module_name> with the name of your module.  
    * Replace URL with the upload URL you got back from the curl command in Step 3.  
    * **The URL is provided in the upload links attribute in the registry-module-versions resource**.  
* **NOTE**: This curl command will not give any output.  
  
## **Viewing Your Module in the TFE Private Module Registry**  
  
* If the module was successfully uploaded, you will be able to see it in the TFE UI by selecting the **Modules** menu or navigating to `https://<tfe_server>/app/<organization>/modules`.  
* Where `<tfe_server>` is the URL of your TFE server and `<organization>` is the organization on that TFE server into which you loaded the module.  
* You can then click the **Details** button to see the module including instructions for using it from Terraform code.  
* You can also see a list of your modules using curl.  
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
      https://app.terraform.io/api/v2/registry-modules/actions/delete/pphan/my_module  
  
* Replace "**pphan**" with your org name  
* Replace "my_module" with your module name.  
  
[1]: Source:%20https://www.terraform.io/docs/cloud/api/modules.html  
[2]: https://www.terraform.io/docs/enterprise/registry/index.html  
[3]: https://www.terraform.io/docs/enterprise/api/modules.html  
[4]: https://www.terraform.io/docs/enterprise/api/modules.html#create-a-module  
[5]: https://www.terraform.io/docs/enterprise/api/modules.html#sample-payload-1  
[6]: https://www.terraform.io/docs/cloud/api/modules.html#create-a-module-version  
[7]: https://www.terraform.io/docs/enterprise/api/modules.html#create-a-module-version  
[8]: https://www.terraform.io/docs/enterprise/api/modules.html#sample-payload-2  
