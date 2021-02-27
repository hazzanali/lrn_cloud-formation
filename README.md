# CloudFormation
AWS CloudFormation is a service that uses the IaC(Infrastructure as a Code) to automate the provisioning of AWS resources. It is synonymous to Microsoft Azure's Azure Resource Manager.

Benefits:
- Reliable and Repeatable.
- Better trackability when used with Source Control
- No more environment drift
- Easy and faster to spin up new environments

CloudFormation template can be written in either JSON or YAML. The output of a successfully run CloudFormation template is a Stack(in Azure ResourceGroup) and all the resource created by the template are associated to the created stack.

# Structure of CloudFormation template
The structure of the CloudFormation template is defined below the order doesn't really matter but it helps to follow a logical sequence to help with readability.

A bare-minimum CloudFormation template must contain atleast the Resources.

1. **AWSTemplateFormationVersion** - At the time of writing, this is 2010-09-09. Maybe in the future Amazon can introduce a new version.

1. **Description** - This is where a good description that explains what your stack does goes.

1. **Metadata** - An optional section to include random objects that more information about the resource. Some features use this section to do advanced configuration.

    ```YAML
    WebServer:
        Type: AWS::EC2::Instance
        Metadata:
            'AWS::CloudFormation::Init':
                configSets:
                wordpress_install:
                - install_cfn
                - install_wordpress
                - configure_wordpress
    ```

1. **Parameters** - gives the CloudFormation template a bit of dynamicity to facilitate reuse. Due to CloudFormation quotas, upto 200 parameters can be created.

    ```YAML
    Parameters:
        InstanceType:
            Description: EC2 instance type for the WebServer
            Type: String
            Default: t2.micro
            AllowedValues:
            - t1.micro
            - t2.nano
            - t2.micro
            - t2.small
            - t2.medium
            - t2.large
            ConstraintDescription: Must be a valid EC2 instance type.
    ```

    Creating constraints can be achieved using the following
    - *AllowedValues* - a list/array of allowed values that can be choosen from a dropdown.
    - *AllowedPattern* - can write regular expression to guard the input.
    - *ConstraintDescription* - the text to be displayed when constraints aren't met.

1. **Mappings** - are similar to a lookup/dictionary of object values. It needs a Key and Name-Value pair. 200 mappings is the upperlimit supported by CloudFormation. To access mapping use the in-built function `!FindInMap[MappingDemon, KeyOne, NameAlt]`

    ```YAML 
    MappingDemo:
        KeyOne:
            Name: ValueOne
            NameAlt: ValueOneAlt
        KeyTwo:
            Name: ValueTwo
            NameAlt: ValueTwoAlt
        KeyThree:
            Name: ValueThree
            NameAlt: ValueThreeAlt
    ```
1. **Conditions** - a section to create boolean conditional that can be used to conditional create resource or add properties.

1. **Transform** - the transform section allows you to specify external macros(a small utility routine) written in either JSON or YAML and stored in S3 bucket. There are two types of transforms; self-defined and managed.

1. **Resources** - the only required section where the resources to be created are configured. CloudFormation supports a maximum of 500 resources.

1. **Outputs** - this section can be used make a resource or its property exposed outside of the stack. The required properties are *Name* and *Value*. 

    ```YAML
    Outputs:
    WebsiteUrl:
        Value: !Join
        - ''
        - - 'http://'
            - !GetAtt
            - WebServer
            - PublicDnsName
            - /wordpress
        Description: Wordpress Website URL
    ```
    The outputs can be exported for wider use across many stacks.However, once exported, they cannot be modified or deleted until all the reference to other stacks are terminated. 

# Methods to update CloudFormation
There are two methods to update stacks in CloudFormation; direct updates and change sets.

## Direct updates
- changes are updated and deployed immediately.
- are used for changes that are to be applied immediately.
- are risky as they don't check for any impacts on resource. For example, it will go ahead and replace a resource without any form of warnings.

## Changesets
- allow us to preview the changes that the update template would make.
- provides a JSON formatted preview of the change summary.
- can help avoid unintended changes to resources.

# Nested stacks
`Important!` Nested stacks are stacks that are created by another stack using the AWS type;AWS::CloudFormation::Stack.

- Nested stacks can contain other nested stacks(heirarchy).
- The top-level stack is called the root stack.
- Each nested stack has a parent.
- Any updates made to the nested stack should be propagated via the root stack.
- Nested stacks live in S3 bucket.
- The stacks are created from the root and if there are two independent stacks CF would execute them at the same time.
- AWS Best practice recommends using Nested Stacks to declare common components.

# Control features
AWS CloudFormation provides various control features to manage and add guards to protect and control your resources.

## Stack Policy
Stack policy help protect sensitive resources from being accidently modified.

Stack policy is a JSON document that prevents updates or deletion of sensitive resources.
- Stack policies protect all resource by default and permission should be explicitly granted.
- One stack policy per stack, can be used to protect multiple resource.
- Only applies during updates and not creation.

> Stack policies are only a fail-safe mechanism, use AWS best practice to create an IAM role to update stacks.

```JSON
{
"Statement" : [
    {
    "Effect" : "Allow",
    "NotAction" : [ "Update:Delete", "Update:Replace" ],
    "Principal": "*",
    "Resource" : "*"
    }
]
}
```

```JSON
{
  "Statement" : [
  {
    "Effect" : "Deny",
    "Principal" : "*",
    "Action" : "Update:*",
    "Resource" : "*",
    "Condition" : {
      "StringEquals" : {
        "ResourceType" : ["AWS::EC2::SecurityGroup"]
      }
    }
  },
  {
    "Effect" : "Allow",
    "Principal" : "*",
    "Action" : "Update:*",
    "Resource" : "*"
  }
  ]
}
```

## Rollback configuration
Rollback configuration allows you configure CloudWatch alarm to rollback changes made to the stack.

For instance, you can create a Rollback configuration that can rollback the recent changes if a CloudWatch alarm for healthcheck gone off. Only IN_ALARM state is considered.

- **Monitoring time**

    Number of minutes after the update operation completes that CloudFormation should monitor the specified CloudWatch alarm.

- **CloudWatch alarm**

    ARN of the alarm to monitor.

## Notification options
With notification options you can create a topic within AWS SNS to send notification on the stack update process.

## Rollback on failure
Toggle that controls whether the stack should be rolled back if stack creation fails.

## Terminiation protection
Toggle that protects stacks from being accidently deleted. When enabled the user is shown a dialog that tell the user that Termination protection has been enabled.

## Cancelling an update
It is possible to cancel updates but there is chance that CloudFormation update may fail to rollback.