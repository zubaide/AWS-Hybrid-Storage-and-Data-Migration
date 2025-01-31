# Hybrid Storage and Data Migration with AWS Storage Gateway File Gateway
## Introduction
### In this tutorial, we will use the AWS Storage Gateway File Gateway service to attach a Network File System (NFS) mount to an on-premises data store. We will then replicate that data to an S3 bucket in AWS. Additionally, we will configure advanced Amazon S3 features, like Amazon S3 lifecycle policies and cross-Region replication.

### After completing this tutorial, we should be able to:
1. Configure a File Gateway with an NFS file share and attach it to a Linux instance
2. Migrate a set of data from the Linux instance to an S3 bucket
3. Create and configure a primary S3 bucket to migrate on-premises server data to AWS
4. Create and configure a secondary S3 bucket to use for cross-Region replication
5. Create an S3 lifecycle policy to automatically manage data in a bucket

This tutorial environment uses a total of three AWS Regions. A Linux EC2 instance that emulates an on-premises server is deployed to the us-east-1 (N. Virginia) Region. The Storage Gateway virtual appliance is deployed to the same Region as the Linux server. In a real-world scenario, the appliance would be deployed in a VMware vSphere or Microsoft Hyper-V environment, or as a physical Storage Gateway appliance.

The primary S3 bucket is created in the us-east-2 (Ohio) Region. Data from the Linux host is copied to the primary S3 bucket. This bucket can also be called the source.

The secondary S3 bucket is created in the us-west-2 (Oregon) Region. This secondary bucket is the target for the cross-Region replication policy. It can also be called the destination.

![image](https://github.com/user-attachments/assets/e5df56d7-4ad4-4a1f-9f8a-59eb26f151af)
This is the network diagram for this project.

## Let's create the Primary and secondary S3 buckets
Before we configure the File Gateway, we must create the primary S3 bucket (or the source) where we will replicate the data. We will also create the secondary bucket (or the destination) that will be used for cross-Region replication.

In the search box to the right of  Services, search for and choose S3 to open the S3 console.

Choose Create bucket then configure these settings:
        Bucket name: Create a name that you can remember easily. It must be globally unique.
        Region: US East (Ohio) us-east-2
        Bucket Versioning: Enable
         _For cross-Region replication, you must enable versioning for both the source and destination buckets._ 

Choose Create bucket

Repeat the previous steps in this task to create a second bucket with the following configuration:
        Bucket name: Create a name you can easily remember. It must be globally unique.
        Region: US West (Oregon) us-west-2
        Versioning: Enable

![image](https://github.com/user-attachments/assets/27ef28d3-57d9-48af-b26b-4cdf38e243e4)
Two S3 bucket in different regions
