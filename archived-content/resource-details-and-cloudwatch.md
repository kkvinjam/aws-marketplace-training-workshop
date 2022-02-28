---
weight: 4
title: "Resource details and CloudWatch"
---

**Important: this lab builds on the components deployed in [Config rule with remediation](../configrule-with-remediation). You will need to deploy the stack from that lab to continue below.**

#### Ensure the CloudWatch agent is installed on instances
 
Now we will create a State Manager job that will run on a schedule to make sure the latest version of the CloudWatch agent is installed on our instance. State Manager is a secure and scalable configuration management service that ensures your Amazon EC2 and hybrid infrastructure is in an intended or consistent state.
 
1. Go to the Systems Manager Console, and on the left side under *Actions* click on **State Manager**. 
2. Click on **Create Association** 
3. Enter ```CloudWatchAgentInstall``` as the association name
4. Under Document, click the radio button next to ```AWS-ConfigureAWSPackage``` command
5. Under Parameters 
      - Action: *Install*
      - Installation type: *In-place update*
      - Name: *AmazonCloudWatchAgent*
6. Under **Targets**, select *Choose instances manually*, and then check to select the EC2 instance that you craeted earlier in the lab. 
      - If this is not a shared or pre-existing AWS account then there is likely only a single instance in the list
7. Under Specify schedule 
      - Select Radio Button next to CRON schedule builder 
      - Enter *Every Day at 22:30*
8. Click on **Create Association** 

![AWS Config State Manager Association](../images/configssmassociation.png)
 
This association will run every day at 10:30 PM and make sure the latest version of the CloudWatch Agent is installed. We can then run another association to pull down the configuration from the Parameter Store 
 
#### What did we learn? 
 
  - How to use State Manager to mensure that the CloudWatch agent is installed and up to date
  - We can use State Manager to ensure a certain state of our EC2 instances 

#### Observe the configuration and compliance timelines
 
We can view the timeline of compliance and configuration changes for resources inside of your AWS environment from directly within the Config console. To do so, follow these steps:
 
1. Go to AWS Config, and click on **Resources**
2. Under the *Resource type* section, type ```AWS EC2 Instance```
3. Click into the instance we deployed earlier in this lab, and then click on **Resource timeline**:

![](../images/configtimeline.png) 

4. Observe the timeline, and the changes that occurred - most especially the changes. Note that every change, including compliance evaluations, are reflected here.:

![](../images/ConfigConfigurationTimeline.png)

5. Expand the non-compliant finding and then click on **View full record** to see the details for this resource at the time the evaluation took place.

![](../images/ConfigConfigurationTimeline2.png)

6. Then click on the EC2 instance's **resource ID**:

![](../images/ConfigConfigurationTimeline3.png)

7. From here you can see the other AWS resources that share a relationship with your EC2 instance. This includes elastic IP addresses, security groups, subnets, VPCs, and storage volumes:

![](../images/ConfigConfigurationTimeline4.png)

**Config maintains an internal database of relationships between resources in your environment. This lab only shows the basic interactions you can have with the Config resource database.**

#### CloudTrail and CloudWatch Logs Insights

Our final step is to use Cloudwatch Logs Insights to investigate the volume of activity created by our work in the lab today. CloudWatch Logs Insights is a powerful query engine that an analyze activity across CloudWatch log groups, and our labs have generated a great deal of activity that we can investigate.

1. Navigate to the Amazon CloudWatch Logs console, and then from the left-side click on **Insights**.
2. Choose the **Log Group** you want to work with from the drop down, and then select **management-tools-week**.
3. Choose to define a relative time for the past three hours.
4. On the right, click on **Queries**, then expand **Sample queries**, **CloudTrail**, **Number of log entries by service, event, and region**, and finally click **Apply**.

![CW Insights ManagementToolsWeek/CloudTrail](../images/cloudwatchinsights.png)

5. Click **Run query**

To limit your search to only activity related to Config, modify your query to reflect this:

```
fields @message
| filter eventSource = 'config.amazonaws.com'
```

This completes the basic labs for Config. From here, you can clean up resources created by visiting [the cleanup steps](../cleanup).