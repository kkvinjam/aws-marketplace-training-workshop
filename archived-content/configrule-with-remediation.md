---
weight: 2
title: "Config rule with remediation"
---

#### Create the lab environment

Before proceeding, you must create a CloudFormation stack that includes the resources required for this lab. [**Click here to deploy this stack into your account:**](https://console.aws.amazon.com/cloudformation/home#/stacks/new?region=us-east-1&stackName=ConfigSSMLab&templateURL=https://s3.amazonaws.com/workshop.aws-management.tools/config/template/ConfigSSMLab.yml) 

[![Create stack](../images/launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?region=us-east-1&stackName=ConfigSSMLab&templateURL=https://s3.amazonaws.com/workshop.aws-management.tools/config/template/ConfigSSMLab.yml)

The stack will create these resources for you:

![](../images/ConfigSSMLab.png)

**Note:** This link will take you to the us-east-1 AWS region. If you wish to use another region, you will need to adjust the region in the top right-hand corner of the console.

**Note:** Make sure to specify the same S3 bucket name in the **Bucket** parameter as is assigned to the CloudTrail trail from the [Setup](../setup) section.
 
#### Creating a Config rule to alert on Systems Manager agent non-compliance
 
In this step we will create a Config rule that will evaluate if EC2 instances have a working Systems Manager agent. 
 
1. Go to the AWS Config console, and then click on **Rules** on the left side of the console. 
2. Click on **Add Rule** 
3. In the Add Rule screen in the Filter section type ```ec2-instance-managed-by-systems-manager```, click on the **ec2-instance-managed-by-systems-manager** rule. 
4. Under the Trigger Section take notice of the trigger type. Leave the remaining settings as-is.  
 ![](../images/ssmagentconfigrule1.png)
 ![](../images/ssmagentconfigrule2.png)
5. Click **Save**
 
You can create config Rules to monitor a number of items within your infrastructure. Beside utilizing AWS managed Config rules you can also create custom rules using AWS Lambda functions. Located here in [Github](https://github.com/awslabs/aws-config-rules) are same sample config rules you can create and implement in Lambda. 
 
#### **Deploy an EC2 instance** 
 
Next, let's deploy and EC2 instance to test our Config rule. Note that we are **not** assigning an IAM role to the instance - that comes later! 

There are two ways to do this:

1. You can do this easily from the EC2 console. Create a ```t3.small``` instance in the same region, with no keypair or IAM instance profile. The instance should use Amazon Linux 2 as the base image, and all default options should be sufficient for creating our lab instance.

Or...

2. Or you can run the following command from the AWS CLI using this command:

``` 
aws ec2 run-instances --image-id $(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text) --count 1 --instance-type t3.small --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=configruletest}]' 
```

The instance should be up and running in approximately one minute.

Now return to the Config rule you created, click into the rule, and click **Re-evaluate** after the instance is up and running. You will have wait a minute or two for the result, and then refresh the web page. After a few moments the instance we deployed should be flagged as non-compliant.
 
![](../images/reevaluatessmrule.png)

Will then look like this:

![](../images/reevaluatessmrule2.png)
 
Next you will fix this non-compliant resource by adding a remediation action to the Config rule.
 
1. Go back to the Config console, and edit the ```ec2-instance-managed-by-systems-manager``` rule. We will set a remediation action to attach a required IAM Role. Select **Actions** and then under **Choose remediation action** do the following:

    - Remediation method: ```Manual remediation```
    - Remediation action: ```AWS-AttachIAMToInstance```
    - Resource ID parameter: *InstanceId* 
      - This passes the non-compliant instance ID to the remediation action 
    - Get the IAM Role name from the output of the CloudFormation stack. The parameter is named ```EC2RoleName```. Enter this into the *RoleName* field.

2. Click **Save** 

![](../images/ssmremediation1.png)
![](../images/ssmremediation2.png)

3. Go back into the Config rule and look at non-compliant resources. Select the instance we deployed and then click on **Remediate**.

![Remediate Button](../images/remediatebutton.png)

4. Visit the Systems Manager console, and then click on **Automation** on the left side. You should see an automation task begin, and this will attach the IAM role to the instance.

![Systems Manager Automation](../images/automation.png)

5. Once completed, reboot the instance to hasten the remediation process. This will force the Systems Manager agent on the instance to acquire the new IAM role immediately upon reboot.
6. Return to the Systems Manager console, and then check under **Managed instances**. When the instance shows up as a managed instance, re-evaluate the rule once more. You will see that the instance is now compliant. 
 
#### What did we learn? 
 
  - How to create an AWS Config Rule to evaluate if instances are managed by SSM 
  - How to use AWS Systems Manager Automation Documents to remediate non-compliant instances 
 
 For our next lab we will use an alternative approach to remediating a non-compliant resource. Click on [Config rule with Lambda](../configrule-with-lambda) to explore a more customizable and extensible method of using Config.
