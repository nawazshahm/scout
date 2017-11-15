## Introduction

Scout2 rules use a recursive engine and a battery of test cases, which means no Python or coding skills are necessary to create or modify a rule; however, understanding of Scout2's data structure may be necessary if you're starting from scratch.

## 1. Simple rule
All Scout2 rules are defined under `AWSScout2/rules/data/findings/`. The following snippet is the entire definition of the `User without MFA` rule. Let's have a look at the various values that are defined to make it work:

* description: A brief description of the rule, displayed on the service's dashboard.
* path: The path to the resources for which the rule applies. `id` is used when iteration over all items in a list or dictionary should be applied.
* dashboard_name: The human-friendly name of the resources that are checked, displayed on the service's dashboard.
* conditions: A list of conditions that, if met, will result in the resource to be flagged.
* id_suffix: If a custom HTML view has been created, this is used to enable color highlighting.

```
{
    "description": "User without MFA",
    "path": "iam.users.id",
    "dashboard_name": "Users",
    "conditions": [ "and",
        [ "iam.users.id.", "withKey", "LoginProfile" ],
        [ "iam.users.id.MFADevices", "empty", "" ]
    ],
    "id_suffix": "mfa_enabled"
}
```

At the very least, these five attributes must be set when creating a new rule for Scout2. 

## 2. Conditions

As mentioned above, the `conditions` attribute is a list of conditions that must be met in order for the processing engine to flag the resource as a finding. The basic format of a condition expression is as follow:

1. The first element may be string that declares the logical operation that must be performed when combining each condition's result when multiple conditions are necessary; its value must be one of "and" or "or".

1. Any other element must be a list of 3 items:
   1. The path to the value to be tested
   1. The test case to be used (provided by opinel)
   1. The value to be tested against

If a rule can be summarized by a single condition, it may be defined as follow:

```
 "conditions" = [ "path_to_value_1", "equal", "trigger_value_1" ]
```

If multiple conditions are necessary, then the format would look as follow:

```
"conditions" = [
    "and",
    [ "path_to_value_1", "equal", "trigger_value_1" ],
    [ "path_to_value_2", "equal", "trigger_value_2" ]
]
```

Note that conditions may be nested, so a more complex rule may look like the following:

```
"conditions" = [
    "and",
    [ "path_to_value_1", "equal", "trigger_value_1" ],
    [ "path_to_value_2", "equal", "trigger_value_2" ],
    [
        "or",
        [ "path_to_value_3", "equal", "trigger_value_3" ],
        [ "path_to_value_4", "equal", "trigger_value_4" ]
    ]
]
```

To avoid code duplication, conditions that are used in multiple rules may be declared in standalone files and included in a rule. In this case, the definition of the rule's conditions would look as follow.

```
"conditions" = [
    "and",
    [ "path_to_value_1", "equal", "trigger_value_1" ],
    [ "_INCLUDE_(conditions/condition-filename.json)", "", ""]
```

Where the files stored under `AWSScout2/rules/data/conditions/condition-filename.json` contains the following payload:

```
{
    "conditions": [ "path_to_value", "testCase", "test_value" ]
}
```

#### Defining dynamic test values

Sometimes, it can be necessary to perform tests against dynamic values. The \_GET\_VALUE\_AT\_ macro can help to parameterize the rule at runtime. For example, the rule defined in `AWSScout2/rules/data/findings/ec2-security-group-opens-all-ports-to-self.json`, which flags security groups that create a virtually flat network between all instances associated with the group, is defined as follow:

```
{
    "description": "Unrestricted network traffic within security group",
    "path": "ec2.regions.id.vpcs.id.security_groups.id.rules.id.protocols.id.ports.id.security_groups.id",
    "dashboard_name": "Rules",
    "conditions": [ "and",
        [ "_INCLUDE_(conditions/security-group-opens-all-ports.json)", "", ""],
        [ "ec2.regions.id.vpcs.id.security_groups.id.rules.id.protocols.id.ports.id.security_groups.id.GroupId", "equal", "_GET_VALUE_AT_(ec2.regions.id.vpcs.id.security_groups.id)" ]
    ],
    "display_path": "ec2.regions.id.vpcs.id.security_groups.id"
}
```

Notice the use of the shared condition "security-group-opens-all-ports" in this rule. The second condition checks whether one of the security group grants authorizes an EC2 security group whose ID matches the source group ID. This is enabled by the use of \_GET\_VALUE\_AT\_

#### Defining dynamic source values

Some rules may require knowledge of the ID associated with different type of resources and combine them in order to fetch the right test value. Similar to test values, the \_GET\_VALUE\_AT\_ command may be used to provide a parameterized path to the source value at runtime. For example, matching EC2 instances with network ACL configuration can be challenging for the following reasons:

* Network ACLs are defined at a VPC level
* Network ACLs are associated at a subnet level
* EC2 instances inherit their subnet's ACLs

In order to create a rule or finding that flags EC2 instances whose subnet's NACLs are wide open, several calls to \_GET\_VALUE\_AT\_ are necessary, as illustrated in the following snippet:

```
{
    "conditions":
        [ "vpc.regions.id.vpcs.id.network_acls._GET_VALUE_AT_(vpc.regions.id.vpcs.id.subnets._GET_VALUE_AT_(ec2.regions.id.vpcs.id.instances.id.network_interfaces.id.SubnetId).network_acl).allow_all_ingress_traffic", "notEqual", "0" ]
}
```

This snippets illustrates that calls to \_GET\_VALUE\_AT\_ may be nested. In such event, the processing engine resolves the deepest \_GET\_VALUE\_AT\_, then iterates back to the top until all values are resolved. In this example, values would be resolved in this order:

1. The subnet ID associated with the network interface
2. The network ACL ID associated with the previously determined subnet ID

