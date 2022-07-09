# z/OS dev/test VPC Custom Image Builder

This set of scripts and automation can be used in conjunction with Wazi Image Builder to create a custom image for use with [Wazi-aaS Virtual Server Instances](https://www.ibm.com/cloud/wazi-as-a-service), in IBM Cloud. Wazi Image Builder, after selecting a set of z/OS volumes, will upload them to Cloud Object Storage as ECKD files. These scripts can then process these files, split them between boot and data volumes, and create a qcow2 image from the boot volumes. This qcow2 image is then uploaded back to the Cloud Object Storage bucket, to be used to create a custom image in a VPC environment.

## Preparations

1. [Install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
2. Duplicate the Terraform variables template file: `cp my-settings.auto.tfvars-template my-settings.auto.tfvars`
3. Adjust [my-settings.auto.tfvars](my-settings.auto.tfvars-template)
   - set `ibmcloud_api_key=<your API key>`
      - this will likelly require a paying account
      - you can create an API account by visiting the [IBM Cloud API keys page](https://cloud.ibm.com/iam/apikeys). Ensure you have
        selected the account you want to use before creating the key as the key will be associtated to the account you have selected
        at the time of creation.
      - If you have downloaded your `apikey.json` file from the IBM Cloud UI you may use this command:
        `export IC_API_KEY=$(cat ~/apikey.json | jq -r .apikey)`
4. Clone this repo to your local machine
5. Run `terraform init` from within this repo's directory

## Running

1. Use Wazi Image Builder to upload your z/OS image to IBM Cloud Object Store (COS)
2. Adjust [my-settings.auto.tfvars](my-settings.auto.tfvars-template) with the name of the COS bucket
   - **NOTE:** there is currently a bug in the Image builder not uploading the `image-metadata.json` as json. As circumvention
     you can use the COS UI to download it and upload it again. This will correct the format.
3. Run:

   ```bash
   terraform apply
   ```

  This will create the VSI with the required data volumes. You might want to use the VSI serial console: the progress logs are written there by cloud init.

Once compeleted successfully, the following can be observed as output:

- A bootable qcow2 image is uploaded to the IBM Cloud Object Storage bucket
- A VPC block storage device, storing data volumes from the z/OS image, is created

Create the z/OS image from the `wazi-custom-image` qcow2 file in your IBM Cloud Object Storage bucket, and snapshots out of the remaining `wazi-custom-image-data` data volume. **TBD**: this will be done by `terraform apply` in following versions.

Use

```bash
terraform destroy
```

to clean up temp resources (e.g., VSI, boot volume, data volume).

## Using the custom image

### Terraform

**Comming**

### UI

1. Go to the [create VSI UI](https://cloud.ibm.com/vpc-ext/provision/vs)
2. Select `IBM Z` as platform
3. Give the VSI a name
4. Select `Custom image as OS`. By default the custom image will be called `wazi-custom-image`
5. Add a new data volume
   1. Select `Import from Snapshot`
   2. The snapshot is called `wazi-custom-image-data` by default
6. Click on `Create VSI`


## Debugging data mover VSI

- you can ssh into the daza mover VSI with `ssh -i private_key root@<floating IP of the VSI>`
- you can copy the cloud-init log output with `ssh -i private_key root@<floating IP of the VSI> cat /var/log/cloud-init-output.log`