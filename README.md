# Hybrid Storage and Data Migration with AWS Storage Gateway File Gateway

## Introduction  
In this tutorial, we will use the AWS Storage Gateway File Gateway service to attach a Network File System (NFS) mount to an on-premises data store and replicate that data to an S3 bucket in AWS. We'll also configure advanced Amazon S3 features like lifecycle policies and cross-Region replication.

### Learning Objectives  
After completing this tutorial, you should be able to:  
1. Configure a File Gateway with NFS file share attached to a Linux instance  
2. Migrate data from a Linux instance to an S3 bucket  
3. Create and configure primary/secondary S3 buckets for data migration  
4. Implement cross-Region replication between S3 buckets  
5. Create S3 lifecycle policies for automated data management  

### Architecture Overview  
![Architecture Diagram](https://github.com/user-attachments/assets/e5df56d7-4ad4-4a1f-9f8a-59eb26f151af)  
*Three-Region architecture with:*
- Linux EC2 instance (on-premises emulation) in us-east-1 (N. Virginia)  
- Storage Gateway appliance in us-east-1  
- Primary S3 bucket in us-east-2 (Ohio)  
- Secondary S3 bucket in us-west-2 (Oregon)  

---

## Step 1: Create Primary and Secondary S3 Buckets

1. **Navigate to S3 Console**  
   - Search for "S3" in AWS Services and open the console

2. **Create Primary Bucket**  
   - Click **Create bucket**  
   - Configure settings:  
     - **Bucket name**: [Unique name]  
     - **Region**: US East (Ohio) us-east-2  
     - **Bucket Versioning**: Enable  
   - Click **Create bucket**

3. **Create Secondary Bucket**  
   - Repeat above steps with:  
     - **Bucket name**: [Unique name]  
     - **Region**: US West (Oregon) us-west-2  
     - **Versioning**: Enable  

![Bucket Creation](https://github.com/user-attachments/assets/27ef28d3-57d9-48af-b26b-4cdf38e243e4)  
*Two S3 buckets in different regions*

---

## Step 2: Configure Cross-Region Replication

1. **Set Up Replication Rule**  
   - Select primary bucket → **Management** tab → **Create replication rule**  
   - Configuration:  
     - **Name**: crr-full-bucket  
     - **Status**: Enabled  
     - **Scope**: Apply to all objects  
     - **Destination**: Secondary bucket  
     - **IAM Role**: S3-CRR-Role (pre-created)  

2. **Test Replication**  
   - Upload test file to primary bucket:  
     ```bash
     aws s3 cp test-file.txt s3://primary-bucket-name/
     ```
   - Verify replication in secondary bucket  

![Primary Bucket File](https://github.com/user-attachments/assets/5dd1949f-afd3-4c46-9b9e-90313ede4484)  
*File in primary S3 bucket*

![Replicated File](https://github.com/user-attachments/assets/c71318e0-a939-45fb-9023-1dfb550df776)  
*Replicated file in secondary bucket*

---

## Step 3: Configure File Gateway & NFS Share

1. **Deploy Storage Gateway**  
   - Open Storage Gateway Console (us-east-1)  
   - **Create gateway** → **Amazon S3 File Gateway**  
   - EC2 Configuration:  
     - **Instance Type**: t2.xlarge  
     - **VPC**: On-Prem-VPC  
     - **Subnet**: On-Prem-Subnet  
     - **Security Groups**: FileGatewayAccess + OnPremSshAccess  
     - **Storage**: 150GiB EBS volume  

2. **Activate Gateway**  
   - Use EC2 instance's public IP for activation  
   - Configure cache storage after activation  

3. **Create NFS File Share**  
   - **File share protocol**: NFS  
   - **S3 Bucket**: Select primary bucket  
   - **IAM Role**: Use pre-created FgwIamPolicyARN  
   - Note mount command for Linux instance  

---

## Step 4: Mount NFS Share & Migrate Data

1. **Connect to Linux Instance**  
   ```bash
   ssh -i "your-key.pem" ec2-user@ec2-public-ip
2. **Prepare Mount Point**
   ```bash
   sudo mkdir -p /mnt/nfs/s3
3. **Mount File Share**
   ```bash
   sudo mount -t nfs -o nolock,hard <File-Gateway-IP>:/share /mnt/nfs/s3
4. **Verify Mount**
   ```bash
   df -h
5. **Copy Data**
   ```bash
   cp -v /media/data/* /mnt/nfs/s3

## Step 5: Verification & Lifecycle Configuration

1. Verify S3 Migration
   - Check files in primary (us-east-2) bucket
   - Confirm replication in secondary (us-west-2) bucket
4. Create Lifecycle Policy
   - Navigate to primary bucket → Management
   - Create lifecycle rule for Glacier transitions

---

## Conclusion

**We've successfully:**
 - Created multi-region S3 buckets with versioning
 - Implemented cross-Region replication
 - Deployed Storage Gateway appliance
 - Migrated on-premises data to S3
 - Verified data replication

**Next steps could include:**
1. Setting up S3 Intelligent-Tiering for cost optimization
2. Implementing S3 Object Lock for compliance
3. Configuring S3 Access Points for secure access management
4. Implement monitoring with CloudWatch
5. Configure additional lifecycle policies
6. Test disaster recovery procedures

> Note: Remember to delete all resources after testing to avoid unnecessary charges.
