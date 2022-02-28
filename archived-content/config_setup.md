---
weight: 1
title: "Setup"
---

#### Create a trail in AWS CloudTrail
 
AWS CloudTrail is a service that helps you enable governance, compliance, risk auditing, and operational auditing of your AWS account. Actions taken by a principal (typiclally a user, role or AWS service) are recorded as events in CloudTrail. 

To learn more about CloudTrail you can click on this [link](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html). Documentation on creating a Trail via the Console is located [here](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-a-trail-using-the-console-first-time.html#creating-a-trail-in-the-console). 

For today's lab, we will require a new trail to be created.
 
1. Search for the CloudTrail service under the Management Tools Section in the console and click on CloudTrail. 
 
    ![Get to CloudTrail Console](../images/cloudtrailconsole.png) 
 
2. Click on **Getting Started** (if presented with that screen). Once in the CloudTrail Console, click on Trails on the Left Side of the screen. 

3. Then click on **Create trail** to create our trail for this lab. 
 
    ![Create Trail](../images/createtrail.png) 
 
4. Apply the following settings to the trail:

  - Trail name: *management-tools-week*
  - Storage location: *Create new S3 bucket (default)*
  - Trail log bucket and folder: *leave untouched*
  - Log file SSE-KMS encryption: *unchecked*

![Create Trail Configuration](../images/createtrailconfiguration1.png)

  - CloudWatch Logs: *enabled*
  - Log group name: *management-tools-week*
  - IAM Role: *New*
  - Role name: *management-tools-week*

![Create Trail Configuration 2](../images/createtrailconfiguration2.png)

  - Click **Next** to proceed to the next step

5. Click **Create trail** to complete the process
 
We now have a trail capturing activity in our AWS Account. Later on, we will search through our trail to reveal details about our account activity and modifications to our resources.
 
#### Activate Config
 
Config provides a detailed view of the configuration of AWS resources in your account. This includes how the resources are related to one another and how they were configured in the past so that you can see how the configurations and relationships change over time. 

**Note: Config is a regional service. The region where you enable this service must be consistent throughout the labs, otherwise you will have a broken experience. Be sure that the region you perform this operation in remains unchanged throughout the day.**
 
1. Search for the Config Service under the Management Tools Section in the console, and then click on **Config**.
 
    ![Get to Config Console](../images/configconsole.png) 
 
2. Click on **Get started**, and we will follow the setup wizard.

    ![Config Setup Page](../images/config-signup1.png)

3. On the **Settings** page make the following selections

    ![Config Settings](../images/config-signup2.png)

  - Record all resources in this region
  - Include global resources
  - Create AWS Config service-linked role
  - Create a bucket (and accept the default bucket name)

4. Click **Next** on the next screen, bypassing rule selection. We will setup Config rules in the next steps.

5. On the last screen click on **Confirm**.

We now have AWS Config recording changes for supported resources, and you can proceed to [Config rule with remediation](../configrule-with-remediation).
