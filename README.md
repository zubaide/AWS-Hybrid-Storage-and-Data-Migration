# Hybrid Storage and Data Migration with AWS Storage Gateway File Gateway
## Intorduction
### In this tutorial, we will use the AWS Storage Gateway File Gateway service to attach a Network File System (NFS) mount to an on-premises data store. We will then replicate that data to an S3 bucket in AWS. Additionally, we will configure advanced Amazon S3 features, like Amazon S3 lifecycle policies and cross-Region replication.

After completing this tutorial, we should be able to:

    Configure a File Gateway with an NFS file share and attach it to a Linux instance
    Migrate a set of data from the Linux instance to an S3 bucket
    Create and configure a primary S3 bucket to migrate on-premises server data to AWS
    Create and configure a secondary S3 bucket to use for cross-Region replication
    Create an S3 lifecycle policy to automatically manage data in a bucket
