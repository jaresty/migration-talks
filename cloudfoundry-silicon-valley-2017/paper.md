## If you follow a few simple techniques, you can easily migrate deployments with bosh without downtime.

## There are a predictable kinds of problems that you will run into when trying to migrate deployments.

* downtime
* rejected credentials for a variety of reasons
* rejected certificates
* data loss
* jobs failing to start due to accidental job upgrades

## We migrated separate CF Release and Diego Release deployments to a single CF Deployment.


