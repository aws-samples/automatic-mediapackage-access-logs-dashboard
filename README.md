# AWS MediaPackage Access Logs Analysis Dashboard

This automated solution helps media teams gain valuable insights into traffic patterns and requests reaching their AWS MediaPackage deployment. The dashboard provides comprehensive visualization of both ingress and egress access logs, making it an essential tool for monitoring performance and troubleshooting issues.

## Dashboard Preview

### MediaPackage Ingress Logs Dashboard

![MediaPackage Ingress Dashboard](/Images/Ingress_Access_Logs_1.PNG)

![MediaPackage Ingress Dashboard Details](/Images/Ingress_Access_Logs_2.PNG)

### MediaPackage Egress Logs Dashboard

![MediaPackage Egress Dashboard](/Images/Egress_Access_Logs_1.PNG)

![MediaPackage Egress Dashboard Details](/Images/Egress_Access_Logs_2.PNG)

## Architecture Design 

![Architecture Diagram](/Images/architecture.png)

The solution follows this data flow:

1. AWS MediaPackage Access Logs are sent to Amazon CloudWatch. **Note:** This initial configuration must be performed separately and is not part of this deployment.
2. Logs are streamed to Amazon Data Firehose through CloudWatch subscription filters.
3. Data transformations are applied to the incoming log records by Data Firehose using a Lambda function. The processed data is then buffered and delivered to an S3 bucket in Parquet format for efficient storage and querying.
4. AWS Glue Data Catalog maintains metadata for both Ingress and Egress Access logs tables, enabling Amazon Athena to efficiently query the transformed logs from S3.
5. Amazon QuickSight visualizes the MediaPackage Access Logs through interactive dashboards.

## Deployment Instructions

> **Important:** The S3 bucket where MediaPackage Access logs are stored must be in the same region as Athena to avoid additional costs related to inter-region data transfer.
 
### Step 1: Deploy AWS MediaPackage Access Logs Data Pipeline

1. Sign in to the **AWS Management Console**
2. Navigate to **AWS CloudFormation** > **Create Stack** > **With new resources**
3. Download the AWS CloudFormation template [**`cloudformation/mediapackage_logs_analysis.yaml`**](./cloudformation/mediapackage_logs_analysis.yaml) from this repository
4. In the CloudFormation create wizard, upload the template file
5. For the **`S3BucketName`** parameter, enter the name of an existing S3 bucket where you want to store AWS MediaPackage Access logs
6. Specify a **Stack name** and choose **Next**
7. Leave the **Configure stack options** at default values and choose **Next**
8. Review the details and under **Capabilities**, select the checkbox for **"I acknowledge that AWS CloudFormation might create IAM resources with custom names"**
9. Choose **Submit**

After successful stack creation, the following resources will be deployed:
- 2 Amazon CloudWatch Subscriptions to Kinesis Firehose
- AWS Glue Database
- 4 AWS Glue Tables
- 2 Kinesis Firehose delivery streams
- Lambda function for data transformation
- Required IAM Policies and roles

### Step 2: Set Up Amazon QuickSight

Amazon QuickSight is AWS's Business Intelligence tool that allows you to visualize your MediaPackage access logs data. If you're already using QuickSight, you can skip to Step 3.

1. Log into your AWS Account and search for QuickSight in the list of Services
2. Sign up for QuickSight if you haven't already
3. When prompted, select the **Enterprise Edition**
4. Select **Continue** and complete the account creation process:
   - Choose the AWS region where your S3 bucket containing MediaPackage Access logs is located
5. In the QuickSight permissions section, enable access to **Amazon S3** and select the bucket where your MediaPackage Access logs are stored

### Step 3: Deploy the Dashboards

We'll use the provided YAML templates to deploy both the Ingress and Egress Access logs dashboards in QuickSight.

#### Install the CID-CMD Tool

```bash
pip3 install cid-cmd
```

#### Deploy the Ingress Logs Dashboard

1. Open your terminal and navigate to the directory containing the template files
2. Run the following command to deploy the [ingress logs dashboard template](./dashboards/mediapackage_ingress_logs.yaml):

```bash
cid-cmd deploy --resources ./dashboards/mediapackage_ingress_logs.yaml
```

3. When prompted, provide the following information:
   - For `[dashboard-id]`, press **Enter** to select the default option
   - For `[athena-database]`, press **Enter** to select `mediapackage_logs_db`
   - For `[s3path]`, enter the S3 URI where your Ingress MediaPackage Access logs are stored (e.g., `s3://your-bucket-name/mediapackage-processed-logs/ingress/`)

Upon successful deployment, you'll see a confirmation message with a link to access your dashboard.

#### Deploy the Egress Logs Dashboard

1. In the same terminal, run the following command to deploy the [egress logs dashboard template](./dashboards/mediapackage_egress_logs.yaml):

```bash
cid-cmd deploy --resources ./dashboards/mediapackage_egress_logs.yaml
```

2. When prompted, provide the following information:
   - For `[dashboard-id]`, press **Enter** to select the default option
   - For `[athena-database]`, press **Enter** to select `mediapackage_logs_db`
   - For `[s3path]`, enter the S3 URI where your Egress MediaPackage Access logs are stored (e.g., `s3://your-bucket-name/mediapackage-processed-logs/egress/`)

Upon successful deployment, you'll see a confirmation message with a link to access your dashboard.

## Using the Dashboards

Once deployed, you can access your dashboards through the QuickSight console or via the direct links provided in the deployment output. The dashboards provide:

- Request volume analysis
- HTTP status code distribution
- Geographic distribution of requests
- Client device and browser analytics
- Error rate monitoring
- Performance metrics

## Troubleshooting

If you encounter issues during deployment:

1. Verify that your S3 bucket exists and is in the same region as Athena
2. Check that you have the necessary permissions to create CloudFormation stacks and QuickSight resources
3. Ensure MediaPackage logs are properly configured to send to CloudWatch

## Contributing

Contributions to improve this solution are welcome. Please feel free to submit pull requests or open issues to suggest enhancements.
