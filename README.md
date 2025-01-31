# Hybrid Storage and Data Migration with AWS Storage Gateway File Gateway
## Introduction
### In this tutorial, we will use the AWS Storage Gateway File Gateway service to attach a Network File System (NFS) mount to an on-premises data store. We will then replicate that data to an S3 bucket in AWS. Additionally, we will configure advanced Amazon S3 features, like Amazon S3 lifecycle policies and cross-Region replication.

### After completing this tutorial, we should be able to:
1. Configure a File Gateway with an NFS file share and attach it to a Linux instance
2. Migrate a set of data from the Linux instance to an S3 bucket
3. Create and configure a primary S3 bucket to migrate on-premises server data to AWS
4. Create and configure a secondary S3 bucket to use for cross-Region replication
5. Create an S3 lifecycle policy to automatically manage data in a bucket

**This tutorial environment uses a total of three AWS Regions. A Linux EC2 instance that emulates an on-premises server is deployed to the us-east-1 (N. Virginia) Region. The Storage Gateway virtual appliance is deployed to the same Region as the Linux server. In a real-world scenario, the appliance would be deployed in a VMware vSphere or Microsoft Hyper-V environment, or as a physical Storage Gateway appliance.**

**The primary S3 bucket is created in the us-east-2 (Ohio) Region. Data from the Linux host is copied to the primary S3 bucket. This bucket can also be called the source.**

**The secondary S3 bucket is created in the us-west-2 (Oregon) Region. This secondary bucket is the target for the cross-Region replication policy. It can also be called the destination.**
