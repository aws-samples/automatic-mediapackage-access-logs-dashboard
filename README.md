# Mediapackage Access Logs Analysis Dashboard

This automated dashboard helps media teams gain insights into traffic and requests reaching their AWS MediaPackage deployment. It can also assist with troubleshooting issues.

## How it looks like
**Mediapackage Ingress Logs Dashboard**

![Mediapackage Ingress Dashbaord](/Images/Ingress_Access_Logs_1.PNG)

![Mediapackage Ingress Dashbaord](/Images/Ingress_Access_Logs_1.PNG)

**Mediapackage Egress Logs Dashboard**

![Mediapackage Egress Dashbaord](/Images/Egress_Access_Logs_1.PNG)

![Mediapackage Egress Dashbaord](/Images/Egress_Access_Logs_1.PNG)

## Architecture Design 

![architecture](/Images/architecture.png)

1. AWS MediaPackage Access Logs are sent to Amazon CloudWatch. NB: This step is not a part of the solution and must be performed separately.
2. These logs are streamed to Amazon Data Firehose through a CloudWatch subscription filter.
3. Data transformations are applied to the incoming log records by Data Firehose using a Lambda function. The buffered output is then sent to an existing S3 bucket in Parquet format.
4. Using AWS Glue Data Catalog to retrieve Ingress and Egress Access logs tables metadata, Athena queries the transformed logs from S3.
5. Amazon Quicksight is then used to visualize the MediaPackage Access Logs.


# Deployment

* Note: The S3 bucket where Mediapackage Access logs are stored has to be in the same region as Athena used in this project to avoid extra costs related to inter-region data transfer.
 
**Step1: AWS Mediapackage Access Logs Data Set Deployment**

1. Sign in to the **AWS Management Console**
2. Navigate to the **AWS CloudFormation **console >** Create Stack** > **“With new resources” **

* Note: the template will ask you to enter an S3 bucket name. The S3 bucket where access logs will stored has to be created previously.

3. Download the AWS CloudFormation template **"MediaPackage_Logs_Analysis_CFM_Template.txt"** file from this git repository. In the create wizard specify the **MediaPackage_Logs_Analysis_CFM_Template.txt**  template.
4. fill the following parameter **"S3BucketName"** with a bucket name where you store AWS Mediapackage Access logs.
5. Specify a **“Stack name” **and choose Next
6. Leave the **“Configure stack options”** at default values and choose Next
7. Review the details on the final screen and under **“Capabilities”** check the box for **“I acknowledge that AWS CloudFormation might create IAM resources with custom names”**
8. Choose **Submit**

Once the Stack is created successfully, you will see the following resources deployed:
2 Amazon CloudWatch Subscriptions to Kinesis Firehose, AWS Glue DB, 4 AWS Glue Tables, 2 Kinesis Firehose delivery streams, Lambda function, IAM Policies and roles.


**Step2 : Prepare Amazon QuickSight**

 Amazon QuickSight is the AWS Business Intelligence tool that will allow you to not only view the Standard AWS provided insights into all of your accounts, but will also allow to produce new versions of the Dashboards we provide or create something entirely customized to you. If you are already a regular Amazon QuickSight user you can skip these steps. 

1. Log into your AWS Account and search for QuickSight in the list of Services
2. You will be asked to sign up before you will be able to use it
3. After pressing the Sign up button you will be presented with 2 options, please ensure you select the Enterprise Edition during this step
4. Select continue and you will need to fill in a series of options in order to finish creating your account.
    1. Ensure you select the region that is most appropriate based on where your S3 Bucket is located containing your AWS WAF Logs  report files.
5. Enable the Amazon S3 option and select the bucket where your AWS Mediapackage Access logs are stored 

**Step3 : Deploy the Dashboard**

We will use the **mediapackage_ingress_logs_table-analysis.yaml** and **mediapackage_egress_logs_table-analysis.yaml** templates to deploy the Mediapackage Ingress Access logs and Mediapackage Egresss Access logs dashboards in quicksight. These templates will also create 2 datasets for ingress and egress access logs in Amazon Quicksight.

1. Open you favorite terminal and install the CID-CMD Tool using the following:

`pip3 install cid-cmd`

Let's start with the **mediapackage_ingress_logs_table-analysis.yaml** deployement!

2. Open your favorite terminal, under the directory where you saved **mediapackage_ingress_logs_table-analysis.yaml** template and run the following cli:

`cid-cmd deploy --resources ./mediapackage_ingress_logs_table-analysis.yaml`

3. during the creation you will be prompted to select: 

? [dashboard-id] Please select dashboard to install: [mediapackage-ingress-logs-table-analysis] mediapackage_ingress_logs_table analysis ==>  **Press enter**

? [athena-database] Select AWS Athena database to use: mediapackage_logs_db ==> **Press enter**

? [s3path] Required parameter: s3path (S3 Path for mediapackage_ingress_logs_table table): s3://mediapackage-logs-analysis/mediapackage-processed-logs/ingress/ ==> **change the S3 URI to the S3 Bucket's URI where your Ingress Mediapackage Access logs are stored**

after a successful run , the output will be :

#######
####### Congratulations!
####### mediapackage_ingress_logs_table analysis is available at: https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/mediapackage-ingress-logs-table-analysis
#######


Let's now deploy the **mediapackage_egress_logs_table-analysis.yaml** !

4. Open your favorite terminal, under the directory where you saved **mediapackage_egress_logs_table-analysis.yaml** template and run the following cli:

`cid-cmd deploy --resources ./mediapackage_egress_logs_table-analysis.yaml`

5. during the creation you will be prompted to select: 

? [dashboard-id] Please select dashboard to install: [mediapackage-egress-logs-table-analysis] mediapackage_egress_logs_table analysis ==>  **Press enter**

? [athena-database] Select AWS Athena database to use: mediapackage_logs_db ==> **Press enter**

? [s3path] Required parameter: s3path (S3 Path for mediapackage_egress_logs_table table): s3://mediapackage-logs-analysis/mediapackage-processed-logs/egress/ ==> **change the S3 URI to the S3 Bucket's URI where your Egress Mediapackage Access logs are stored**

after a successful run , the output will be :

#######
####### Congratulations!
####### mediapackage_egress_logs_table analysis is available at: https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/mediapackage-egress-logs-table-analysis
#######

You can now start analysing Mediapackage Access logs...








