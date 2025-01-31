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

1. In the search box to the right of  Services, search for and choose S3 to open the S3 console.

2. Choose Create bucket then configure these settings:
        Bucket name: Create a name that you can remember easily. It must be globally unique.
        Region: US East (Ohio) us-east-2
        Bucket Versioning: Enable
         _For cross-Region replication, you must enable versioning for both the source and destination buckets._ 

4. Choose Create bucket

5. Repeat the previous steps in this task to create a second bucket with the following configuration:
        Bucket name: Create a name you can easily remember. It must be globally unique.
        Region: US West (Oregon) us-west-2
        Versioning: Enable

![image](https://github.com/user-attachments/assets/27ef28d3-57d9-48af-b26b-4cdf38e243e4)
Two S3 bucket in different regions

## Enabling cross-Region replication
Now let's crreate a replication policy

1. Select the name of the source bucket that you created in the US East (Ohio) Region.

2. Select the Management tab and under Replication rules select **Create replication rule**

3. Configure the Replication rule:
   **Replication rule name**: crr-full-bucket
   **Status** Enabled
   **Source bucket:**
       For **Choose a rule scope**, select  Apply to all objects in the bucket
   **Destination**:
       Choose a bucket in this account
       Choose Browse S3 and select the bucket you created in the US West (Oregon) Region.
       Select **Choose path**
       **IAM role**: S3-CRR-Role (This role was pre-created with the required permissions)
         
4. Choose **Save**. When prompted, if you want to replicate existing objects, choose No, and then choose **Submit**
**Note:** there are no objects currently in the bucket, so the answer will have no effect in this case.

5. Return to and select the link to the bucket you created in the US East (Ohio) Region.

6. Choose **Upload** to upload a file from your local computer to the bucket
   _For this lab, use a small file that does not contain sensitive information, such as a blank text file._

8. Choose **Add file**s, locate and open the file, then choose **Upload** 

9. Wait for the file to upload, then choose **Close**. Return to the bucket you created in the US West (Oregon) Region. 

10. The file that you uploaded should also now have been copied to this bucket.
    _Note: You may need to refresh  the console for the object to appear._

![image](https://github.com/user-attachments/assets/4e96c150-0016-4d41-94d5-33b94a3d60a1)

## Configuring the File Gateway and creating an NFS file share
Next task is to deploy the File Gateway appliance as an Amazon Elastic Compute Cloud (Amazon EC2) instance. We will then configure a cache disk, select an S3 bucket to synchronize your on-premises files to, and select an IAM policy to use. Finally, we will create an NFS file share on the File Gateway.

 
1. In the search box to the right of  Services, search for and choose Storage Gateway to open the Storage Gateway console.

2. At the top-right of the console, verify that the current Region is** N. Virginia**.

3. Choose Create gateway then begin configuring the Step 1: Set up gateway settings:
    **Gateway name**: File Gateway
    **Gateway time zone**: Choose GMT -5:00 Eastern Time (US & Canada), Bogota, Lima
    **Gateway type**: Amazon S3 File Gateway
    **Host platform**: choose Amazon EC2. Choose Customize your settings. Then choose the **Launch instance** button.
    A new tab opens to the EC2 instance launch wizard. This link automatically selects the correct Amazon Machine Image (AMI) that must be used for the File Gateway appliance.

4. In the Launch an instance screen, begin configuring the gateway as described:
    **Name:** File Gateway Appliance
    **AMI from catalog**: Accept the default aws-storage-gateway AMI.
    **Instance type**: Select the **t2.xlarge** instance type
    
5. Configure the network and security group settings for the gateway.
    Next to Network settings, choose Edit, then configure: 
      **VPC**: On-Prem-VPC
      **Subnet**: On-Prem-Subnet
      **Auto-assign public IP**: Enable
      Under Firewall (security groups), choose  Select an **existing security group**.
      For Common security groups: 
         Select the security group with FileGatewayAccess in the name
         Note: This security group is configured to allow traffic through ports 80 (HTTP), 443 (HTTPS), 53 (DNS), 123 (NTP), and 2049 (NFS). These ports enable the activation of the File Gateway appliance. They also enable connectivity from the Linux server to the NFS share that you will create on the File Gateway.
         Also select the security group with OnPremSshAccess in the name Note: This security group is configured to allow Secure Shell (SSH) connections on port 22.
         Verify that both security group now appear as selected (details on each will appear in boxes in the console). 
         Tip: You may need to choose Show all selected to see them both.

6. Configure the storage settings for the gateway.
    In the Configure storage panel, notice there is already an entry to create one 80GiB root volume.
    Choose **Add new volume** 
    Set the size of the EBS volume to 150GiB

7. Finish creating the gateway.
    In the Summary panel on the right, keep the number of instances set to 1, and choose **Launch instance**
    A Success message displays.
    Choose **View all instances**
    Your File Gateway Appliance instance will take a few minutes to initialize.

8. Monitor the status of the deployment and wait for **Status Checks** to complete.
    Tip: Choose the refresh  button to more quickly learn the status of the instance.

9. Select your File Gateway instance, then in the Details tab below, locate the** Public IPv4 address** and copy it. 
    You will use this IP address when you complete the File Gateway deployment.

10. Return to the **AWS Storage Gateway** tab in your browser. It should still be at the **Set up gateway on Amazon EC2** screen.

11. Check the box next to I completed all the steps above and launched the EC2 instance, then choose Next

12. Configure the Step 2: **Connect to AWS settings**:
     In the Gateway connection options:
       For IP address, paste in the IPv4 Public IP address that you copied from your File Gateway Appliance instance
     For the Service endpoint, select Publicly accessible.
       Choose Next

13. In the Step 3: Review and activate settings screen choose **Activate gateway**

14. Configure the Step 4: Configure gateway settings:
     CloudWatch log group: Deactivate logging
     CloudWatch alarms: No Alarm
     A Successfully activated gateway File Gateway Appliance message displays. 
     In the Configure cache storage panel, you will see that a message the local disks are loading.  
     Wait for the local disks status to show that it finished processing (approximately 1 minute).
     Choose **Configure**

15. Start creating a file share.
     Wait for File Gateway status to change to Running.
     From the left side panel, choose File shares.
     Choose Create file share.

16. On the Create file share screen, configure these settings:
     **Gateway**: Select the name of the File Gateway that you just created (which should be File Gateway Appliance)
    ** File share protoco**l: NFS
    **Amazon S3 bucket nam**e: Choose the name of the source bucket that you created in the US East (Ohio) us-east-2 Region in Task 1.
     Choose **Customize configuration**
     For File share name use share and choose Next.

17. On the Amazon S3 storage settings screen, configure these settings:
     **Storage class for new objects**: S3 Standard
     **Object metadata**:
        Guess MIME type
        Gateway files acccessible to S3 bucket owner
     Access your S3 bucket: Use an existing IAM role
     IAM role: Paste the _FgwIamPolicyARN_, which you can retrieve by following these instructions –
        Choose the **Details** dropdown menu above these instructions
        Select **Show**
        Copy the FgwIamPolicyARN value
     Choose **Next**

18. In the File access settings screen, accept the default settings.
    Note: You might get a warning message that the file share is accessible from anywhere. For this lab, you can safely disregard this warning. In a production environment, you should always create policies that are as restrictive as possible to prevent unwanted or malicious connections to your instances.

    Choose **Next**

19. Scroll to the bottom of the Review and create screen, then select **Create** 
    Monitor the status of the deployment and wait for Status to change to Available, which takes less than a minute.
    Note: You can choose the refresh  button occasionally to notice more quickly when the status has changed.

20. Select the file share that you just created by choosing the link.

21. At the bottom of the screen, note the command to mount the file share on Linux. You will need it for the next task.


