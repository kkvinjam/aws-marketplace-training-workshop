---
weight: 7
title: "Compliance Management"
---


#### Configuration Compliance

In this lab, we show you how to build a fleet-wide enterprise compliance management and remediation system using [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html), [AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/?id=docs_gateway), and [Amazon CloudWatch](https://docs.aws.amazon.com/cloudwatch/?id=docs_gateway). In addition, we provide compliance stakeholders with visibility into the performance of the compliance system by using [Amazon QuickSight](https://docs.aws.amazon.com/quicksight/?id=docs_gateway) and [Amazon Athena](https://docs.aws.amazon.com/athena/?id=docs_gateway) for reporting.

We will also learn how to [AWS Config Conformance Packs](https://aws.amazon.com/blogs/mt/introducing-aws-config-conformance-packs/) can help you build a configuration compliance solution. 

#### Managing Compliance with AWS Systems Management and AWS Config

In this section, we will use AWS Systems Manager Inventory and AWS Config to catalog all applications installed on an EC2 instance and then blacklist an application.  This application has been deemed as being insecure. **Note:** The application used is only a common sample application, we are not making any statements regarding the security of the application.

#### 7.1 Creating a Non-Compliant EC2 Instance

1. We will first go to the [EC2 Console](https://console.aws.amazon.com/ec2) and click on the **Launch Instance**.
1. Select the **Microsoft Windows Server 2019 Base image**.
1. Select a **General purpose**, **t2.micro instance**, and click **Next: Configure Instance Details**.
1. On the next screen, we will select all the defaults except the IAM role section. In this section, we will select the **AmazonSSMRoleForInstancesQuickSetup** role.
1. We will then click on **Advanced Details** and enter the following in User data.
   ```
   <powershell>
   $url = "https://javadl.oracle.com/webapps/download/AutoDL?BundleId=238698_478a62b7d4e34b78b671c754eaaf38ab"
   $output = "c:\Windows\Temp\JavaSetup8u211.exe"
   $wc = New-Object System.Net.WebClient
   $wc.DownloadFile($url, $output)
   start-sleep 20
   C:\Windows\Temp\JavaSetup8u211.exe /s
   Write-Output "Java is installed"
   </powershell>
   ```
1. Then, click on **Next: Add Storage**.
1. In the Add Storage page, we will go with the default settings and click **Next: Add Tags**.
1. In the **Add Tags** page, we will enter **Name** as the Key and **Java** in the Value sections.
1. Then click on **Review and Launch**.
1. On the **Review Instance Launch** page, click on **Launch**.
1. Then we will select **Proceed without a key pair** from the drop down and click **Launch Instances**. 


#### 7.2 Configuring AWS Systems Manager Inventory

AWS Systems Manager is the mechanism used to gather resource metadata, including custom metadata, such as the location of a server rack for an on-premises managed instance.  The example used in this lab, is simply an example, the same concept can be used for any resource inventory data.  

1. Go to the [AWS Systems Manager Console](https://aws.amazon.com/systems-manager).
1. Then, click on **Managed Instance** in the navigation.
1. Once there, click on the **Setup Inventory** link.
1. In the Name field, we will enter **Inventory-Association**.
1. In the **Targets** section, we will specify the tags we used when launching the EC2 instance.
1. Everything else will be default. Then, click on the **Setup Inventory** button.
1. Once that is configured, we will go to the [Managed Instances](https://us-west-2.console.aws.amazon.com/systems-manager/managed-instances) page.
1. Click on the **Actions** drop down and select **Edit AWS Config recording**.
1. Once on the Settings page, we will make sure **Recording is on**. 


#### 7.3 Configuring the AWS Systems Manager Automation Service Role

When configuring remediation actions within AWS Config, you must provide a service role for Automation to assume during the remedation workflow. To get started quickly, we will use the Automation role created by Quick Setup (AmazonSSMRoleForAutomationAssumeQuickSetup). We will add the **AmazonSSMAutomationRole** AWS managed IAM policy to ensure the role has the appropriate permissions to perform the required actions by the remedation action. For more information on creating an Automation service role, see [Getting started with Automation](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-setup.html).

1. Go to the [AWS  Identity and Access Management (IAM) Console](https://console.aws.amazon.com/iam).
1. Then, click on **Roles** in the navigation.
1. Once there, click on the **AmazonSSMRoleForAutomationAssumeQuickSetup** IAM role.
1. In the **Permissions** tab, select **Attach policies**.
1. On the **Attach Permissions** page, type **AmazonSSM** in the search bar, and select the **AmazonSSMAutomationRole** policy.
1. Choose **Attach policy** and you will be brought back to the IAM roles console page.


#### 7.4 Setting up Config and Config Rules

[AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html) provides a detailed view of the configuration of AWS resources in your AWS account. This includes how the resources are related to one another and how they were configured in the past so that you can see how the configurations and relationships change over time. 

1. Go to the [AWS Config console](https://console.aws.amazon.com/config).
1. From there, we will click on Rules from the navigation.
1. Then, we will click on Add rule.  
1. If prompted, click on the **Get started** button.
1. On the **Settings** page, keep all the default settings and click **Next**.
1. On the **AWS Config Rules** page, click **Next**.
1. Then, on the **Review** page we will click **Confirm**.
1. Click on **Rules** on the left hand navigation.
1. In the **Rules** page, we will click on Add rule page.
1. We will type blacklist in the search bar.
1. We will then select the predefined rule with the name [ec2-managedinstance-applications-blacklisted](https://docs.aws.amazon.com/config/latest/developerguide/ec2-managedinstance-applications-blacklisted.html).
1. Now, we will open a new browser tab to the [Managed Instances](https://console.aws.amazon.com/systems-manager/managed-instances) section of the Systems Manager Console.
1. Then, we will locate the EC2 instance with the tag **Java** and click on the **Instance ID** link.
1. From the instance details page, we will click on the **Inventory** tab the get the software installed on the instance.
1. Copy the exact name of the **Java application** (i.e. `Java 8 Update 211`). We will use this when creating our Config Rules.

    **Note:** If you do not see Java listed in the AWS:Application list, manually re-run the association. Navigate to the State Manager console, select the association for `AWS-GatherSoftwareInventory` and select **Apply association now**.

1. Go back to the **Add AWS managed rule** page.
1. We will configure the **Scope of changes** to **All changes**.
1. We will then enter the appropriate `Java 8 Update 211` version in the Rule parameters configuration.  In the **applicationNames** key, enter the exact Java version copied from the Inventory page (i.e. `Java 8 Update 211`).
1. In the **platformType** key, enter `Windows`.
1. We will then set the **Remediation action** to **AWS-StopEC2Instance**.
1. Set the **Auto remediation** to **Yes**.
1. Set the **Retries** to **1** and **Seconds** to **25**.
1. In the **Resource ID parameter** we will select **InstanceID** from the drop down.
1. Under **Parameters**, we will enter the ARN of the **Automation Service Role** created by the Quick Setup.  Click [here](https://console.aws.amazon.com/iam/home#/roles/AmazonSSMRoleForAutomationAssumeQuickSetup) to get the ARN of the Role.
1. Then we will click on the **Save** button.
1. Once the rule is created, it will take some time for the evaluation to complete.
1. Once the evaluation is complete, we should see 1 noncompliant resource.
1. If we click on the rule name (**ec2-managedinstance-applications-blacklisted**) we will be able to see additional details, including the resource that is not compliant.
1. From the **Rule details** page, we will select the instance that needs remediation and click on the **Remediate** button.
1. At this point, we will see the remediation action being executed.
1. To ensure the action worked, we will go to the [EC2 Console](https://console.aws.amazon.com/ec2) and check whether our instance is shut down or not.


#### 7.5 Monitoring Configuration Changes

Another important aspect of configuration compliance is having the ability to get notification of configuration changes. In this section, we will configure Amazon CloudWatch to send notication messages to Amazon Simple Notification Service (SNS).

In order to receive notifications, an SNS Topic must be created to be used with CloudWatch.

1. Go to the [SNS Console](https://us-west-2.console.aws.amazon.com/sns/v3/home?region=us-west-2#/homepage).  If prompted, click on **Next step**.
1. On the **Create topic** page. Type **Compliance** under **Name** and **Display Name**.
1. Leave all options default and click on **Create topic**.
1. Once a topic has been created, click on **Create subscription**.
1. Select the **Protocol** to be Email and enter a valid e-mail address as the **Endpoint**. Click **Create subscription**.
1. The subscription will not be available until the it is confirmed using the e-mail sent to the e-mail address entered in the Enpoint property above.
1. Once the verification is complete, you will receive a confirmation. 


#### 7.6 Configuring Amazon CloudWatch

CloudWatch is required to be configured in order to identify and detect compliance changes, as well as get notifications of those changes.

1. Go to the [CloudWatch Console](https://console.aws.amazon.com/cloudwatch).
1. Click on **Rules** under **Events**.
1. Click on **Create rule**.
1. In the **Event Source** section, populate the following attributes:
   * **Event Pattern** drop down, select events by service.
   * Service Name = **Config**.
   * Event Type = **Config Configuration Item Change**.
1. In Targets section, click **Add target**, populate the following:
   * Targets = **SNS topic**.
   * Topic = **Compliance**.
1. Leave the rest of the settings default. Click **Configure details**.
1. Under Name type in **ConfigChanges** and click **Create rule**.


#### 7.7 Reporting with Amazon Athena and Amazon QuickSight

The objective of this section is to show you how to identify any systems that are not following security compliance policies. By leveraging both desired state configuration and AWS Config and AWS Config Rules, we can setup a multi-layered approach to ensure our security policies are being followed. Although in this labe we used EC2 instances, because of their simplicity to manage and to also show the before and after, the capabilities highlighted can be used to interact with other services.

In order to use [Athena](https://docs.aws.amazon.com/athena/latest/ug/what-is.html) and [QuickSight](https://docs.aws.amazon.com/quicksight/latest/user/welcome.html), we need to complete some additional pre-requisite operations. This includes creating an S3 bucket and create a policy to allow Systems Manager to connect to the bucket.

1. Go to the [S3 Console](https://s3.console.aws.amazon.com/s3/home?region=us-west-2) and create a bucket. Alternatively, you can use an existing bucket.
1. Then we will apply the following S3 Bucket Policy to allow Systems Manager to access S3.

      **Important:** Make sure you replace ```bucket-name``` with the S3 bucket name and replace ```012345678910``` with your AWS account ID in the policy. To get your account ID, click on the [Support Center](https://console.aws.amazon.com/support/home?region=us-west-2#/) link. It will be displayed on the left hand side of the screen.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SSMBucketPermissionsCheck",
            "Effect": "Allow",
            "Principal": {
                "Service": "ssm.amazonaws.com"
            },
            "Action": "s3:GetBucketAcl",
            "Resource": "arn:aws:s3:::bucket-name"
        },
        {
            "Sid": " SSMBucketDelivery",
            "Effect": "Allow",
            "Principal": {
                "Service": "ssm.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::bucket-name/*/accountid=012345678910/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "bucket-owner-full-control"
                }
            }
        }
    ]
}
```

3. Once the policy has been applied to the bucket, we will configure the inventory data sync.

#### 7.8 Configuring Resource Data Sync for Inventory

1. Go to the [Systems Manager Console](https://console.aws.amazon.com/systems-manager), from the navigation pane select **Inventory**.
1. Then, we will click on the **Resource Data Syncs** button.
1. Once there, we will click on **Create resource data sync** button.
1. We will name type a name for the *Sync name* (any name will do), provide the bucket name of the bucket we previously created. The bucket name will follow a format similar to `dsc-demo-012345678910-us-west-2`. **Important:** Your bucket name will include your account ID and region where the CloudFormation template was launched.
1. Leave all other settings set to default. Click **Create**.


#### 7.9 Using Amazon Athena

In order to access the inventory data from S3, we must first create a schema using Athena. Once the schema is created, we will be able to use it to visualize the data using QuickSight.

1. Open the Amazon [Athena Console](https://us-west-2.console.aws.amazon.com/systems-manager/home?region=us-west-2).
1. If you have never used Athena, we will click on the **Get Started button**.
1. We will then create a database for our compliance data. 

   ```
    CREATE DATABASE dsc
   ```

1. We will then create a table within the `dsc` database by running the following query.  

      **Important:** Ensure you replace `bucket-name` with the name of your S3 bucket before executing.

   ```
    CREATE EXTERNAL TABLE IF NOT EXISTS dsc.status (
    `Status` string,
    `InstalledTime` string,
    `ExecutionType` string,
    `PatchSeverity` string,
    `Title` string,
    `Severity` string,
    `ExecutionTime` string,
    `ComplianceType` string,
    `Classification` string,
    `Id` string,
    `DocumentVersion` string,
    `PatchState` string,
    `PatchBaselineId` string,
    `DocumentName` string,
    `PatchGroup` string,
    `ExecutionId` string,
    `resourceId` string,
    `captureTime` string,
    `schemaVersion` string
    )
    ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
    WITH SERDEPROPERTIES (
    'serialization.format' = '1'
    ) LOCATION 's3://bucket-name/'
    TBLPROPERTIES ('has_encrypted_data'='false');
   ```

1. When that is complete, we will run the following query:
   
   ```
    SELECT
      status,resourceid,compliancetype,executiontime 
    FROM
      dsc.status
    WHERE
      status = 'COMPLIANT' AND compliancetype = 'Association'
   ```
   
#### 7.10 Creating Compliance Reports

1. To visualize the Inventory and Compliance data, we will use Amazon QuickSight. Let’s open the [QuickSight Console](https://quicksight.aws.amazon.com/).
1. If prompted, complete the activation.
1. Then, we will click on the Connect to another data source or upload a file.
1. Then, we will select **Athena**.
1. We will type a **Data source name** and select **Crate data source**.
1. We will then select the dsc database and the status for the data to visualize. Click on the **Select button**.
1. We will use the Spice engine for visualization.
1. You can now begin visualizing your data.

**Note:**  For complete details on Inventory Data Sync, visit the Systems Manage Documentation page: [Walkthrough: Use Resource Data Sync to Aggregate Inventory Data](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-inventory-resource-data-sync.html).


**End of Lab Exercises** 
 
**Thank you for using this lab.** 
 