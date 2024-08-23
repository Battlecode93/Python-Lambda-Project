# **AWS EBS Volume Compliance Monitoring with Python and Lambda**

## **Project Overview**

This project demonstrates the automation of compliance monitoring within an AWS environment using AWS Lambda, Amazon CloudWatch, and EventBridge. The specific scenario addressed is the automatic conversion of Amazon Elastic Block Store (EBS) volumes from the `gp2` type to the more cost-effective `gp3` type upon volume creation. This ensures that your EBS volumes comply with organizational policies for cost management and performance.

## **Architecture**

- **AWS Lambda:** A serverless function that processes events and modifies EBS volumes as per the organizational policy.
- **Amazon CloudWatch:** Monitors the AWS environment and generates events for specific resource activities.
- **Amazon EventBridge:** Captures the events from CloudWatch and triggers the Lambda function upon the creation of a new EBS volume.

## **Deployment Process**

### **1. Create the Lambda Function**

1. **Go to the AWS Lambda Console:**
   - Create a new Lambda function named `ebs_volume_check`.
   - Use Python 3.x as the runtime environment.

2. **Assign Necessary Permissions:**
   - Attach an IAM role with permissions such as `AmazonEC2FullAccess` to allow the Lambda function to modify EBS volumes.

3. **Add the Python Script:**
   - Insert the following Python code into your Lambda function:

   ```python
   import json
   import boto3

   def get_volume_id_from_arn(volume_arn):
       # Split the ARN using the colon (':') separator
       arn_parts = volume_arn.split(':')
       # The volume ID is the last part of the ARN after the 'volume/' prefix
       volume_id = arn_parts[-1].split('/')[-1]
       return volume_id
       
   def lambda_handler(event, context):
       volume_arn = event['resources'][0]
       volume_id = get_volume_id_from_arn(volume_arn)
       
       ec2_client = boto3.client('ec2')
    
       response = ec2_client.modify_volume(
           VolumeId=volume_id,
           VolumeType='gp3',
       )
   ```

### **2. Set Up CloudWatch/EventBridge Rule**

1. **Navigate to CloudWatch:**
   - Go to the Amazon CloudWatch console and create a new rule.
   - This will direct you to Amazon EventBridge for further setup.

2. **Configure EventBridge Rule:**
   - Set the rule to trigger on EBS volume creation events.
   - Under **Event Source**, choose **EventBridge (CloudWatch Events)**.
   - Define an event pattern to capture EBS volume creation events.
   - Set the target of this rule to the `ebs_volume_check` Lambda function.

### **3. Test the Setup**

- **Create a New EBS Volume:**
   - Go to the EC2 console and create a new EBS volume of type `gp2`.
   - Ensure that the volume is created in a region that your Lambda function monitors.

- **Verify Lambda Execution:**
   - After the volume creation, the CloudWatch rule should trigger the Lambda function.
   - Check the EC2 console to verify that the volume type has been automatically converted to `gp3`.

### **4. Monitor and Scale**

- **Monitor Lambda Logs:**
   - Use CloudWatch logs to monitor the execution of the Lambda function.
   - Ensure there are no errors and that all volume modifications are successful.

- **Scaling Considerations:**
   - Consider using AWS Config rules or adding more automation scripts to manage other resources in compliance with your organization’s policies.
   - This approach can be extended to other resource types and policies as required.

## **Summary**

This project exemplifies the use of AWS serverless architecture to enforce compliance policies across your AWS environment. By using AWS Lambda, CloudWatch, and EventBridge, you can automatically manage your EBS volumes, ensuring they conform to your organization's cost and performance guidelines.

