## If you follow a few simple techniques, you can easily migrate your deployments to BOSH 2.0 without downtime.

## You will need to update your BOSH director with a certificate for which you have the CA certificate in hand

In order to use the new BOSH CLI, you need to specify the CA certificate you are using to talk to the BOSH director or you need to use a certificate that has been signed by a trusted certificate authority.  We will show you how to generate and sign your own certificate here, but you will need to insert that certificate into your `bosh-init` manifest and redeploy your director in order for it to work.  Also, the minimum BOSH director version for using BOSH 2.0 schema features is XXXX.

## After migrating to BOSH 2.0, your deployment can take advantage of the following awesomesauce:

These new BOSH features ease the deployment process and make environments more repeatable across multiple IaaSes.

* BOSH links
    - Allow BOSH to share configuration
* BOSH variables
    - Allow you to reuse deployments with per-environment customization injected at deployment time
* BOSH ops files
    - Allow you to customize deployments in a repeatable way
* BOSH cloud config
    - Allow you to separate IaaS-specific configuration from your deployment

## To take advantage of these new BOSH features, BOSH you will need to adapt your manifest as follows:

A deployment manifest should be reusable with no hand-editing across multiple deployments.  You may use BOSH variables to configure required properties, and may wish to use BOSH operations for optional customizations of the base manifest.

BOSH 2.0 schema deployment manifests contain instance_groups instead of templates to mean a "class of virtual machine that can be created and may have multiple jobs on it."  These instance_groups can be stood up in multiple AZs, reducing the repetition that used to be required in a 1.0-schema manifest.

## You will need to create and upload to your new BOSH director:
- A Cloud Config file that contains IaaS-specific information about IP addresses and so on. You should remove all IaaS-specific information from your deployment manifest. You can find an example cloud-config [here](XXX), but you will need to hand-edit yours to match your existing network layout.

## There are a predictable kinds of problems that you will run into when trying to migrate deployments.

* downtime
* rejected credentials for a variety of reasons
* rejected certificates
* data loss
* jobs failing to start due to accidental job upgrades

## We migrated separate CF Release and Diego Release deployments to a single CF Deployment.


