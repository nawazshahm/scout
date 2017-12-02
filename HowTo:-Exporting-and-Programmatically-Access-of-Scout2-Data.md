# Introduction

Being able to export the tool's finding into arbitrary format that may be consumed by other tools, or used to populate internal tracking system, is a feature that has been requested on many occasions and proved useful many times. This wiki page documents how Scout2's data is stored and how it may be exported to other formats.

# 1. Data location and format

After a Scout2 run completes, all the necessary data is stored in a JavaScript file that is included by the HTML report. This file is located under `scout2-report/inc-awsconfig/aws_config-<profile_name>.js`.

This JavaScript file is formatted as follow:

* The first line contains the variable definition code, necessary 
* Lines 2 through EOF contain a JSON-formatted object that contains all the data used by the HTML report, _i.e._ report metadata, aws configuration, and analysis results.

```
aws_info = 
{VALID_JSON_PAYLOAD}
```

Therefore, the entire data set may be used by discarding the file's first line. The following demonstrates several ways in which the data may be programmatically accessed using various common tools.

# 2. Use in Python

The following code snippet illustrates how one may load the Scout2 data in a Python script, assuming this script is in the same folder as the 'scout2-report' folder that contains the Scout2 output.
```
from AWSScout2 import AWSCONFIG
from AWSScout2.output.html import Scout2Report

report = Scout2Report('<profile-name>', 'scout2-report')
aws_config = report.jsrw.load_from_file(AWSCONFIG)
```

# 3. Use with jq

Pretty-print the entire Scout2 data:
```
tail scout2-report/inc-awsconfig/aws_config-l01cd3v.js -n +2 | jq '.'
```

Pretty-print a list of all security groups:
```
tail scout2-report/inc-awsconfig/aws_config-l01cd3v.js -n +2 | jq '.services.ec2.regions[].vpcs[].security_groups[]'
```

# 4. Export to CSV

Scout2 comes with a utility tool that enables exporting of a single finding's resources into comma separated values (CSV). The tool, `Scout2Listall`, uses the same processing engine as Scout2.

The following command line enables listing of all fetched resources with no condition:

```
./Scout2Listall.py --profile <profile-name> --path awslambda.regions.id.functions.id

./Scout2Listall.py --profile <profile-name> --path ec2.regions.id.vpcs.id.security_groups.id
```

If one wanted to list all resources within a given region or VPC ID, the following command may be used:
```
./Scout2Listall.py --profile <profile-name> --path ec2.regions.us-east-1.vpcs.id.security_groups.id

./Scout2Listall.py --profile <profile-name> --path ec2.regions.id.vpcs.vpc-de4977b9.security_groups.id
```

The output of some of these commands only contains a list of the resource IDs; however, any resource attribute may be output. In order to specify which attributes should be output, the `--keys` argument should be used. The keys should contain values formatted as follow:

1. The full path to the attribute (_e.g._ ec2.regions.id.vpcs.id.security_groups.id.name)
2. The path to the attribute relative to the resource being processed (_e.g._ name)

For example, the output of the following commands will be identical:

```
./Scout2Listall.py --profile <profile-name> --path ec2.regions.id.vpcs.id.security_groups.id --keys name

./Scout2Listall.py --profile <profile-name> --path ec2.regions.id.vpcs.id.security_groups.id --keys ec2.regions.id.vpcs.id.security_groups.id.name
```

The `--path` argument may contain a list. For example, the following command will generate a CSV output that lists the AWS region, VPC ID, security group ID, and security group name:

```
./Scout2Listall.py --profile <profile-name> --path ec2.regions.id.vpcs.id.security_groups.id --keys ec2.regions.id ec2.regions.id.vpcs.id ec2.regions.id.vpcs.id.security_groups.id name
```

For convenience, a number of default configurations exist, which is why the output of the tool when working with security groups contains the region, VPC ID, security group ID, and security group name. These configuration files are located under `AWSScout2/output/data/listall-configs/` and are automatically used if available and a different format is not specified.

In order to create your own listall-config file, you may do by creating a file formatted as follow:

```
{
    "keys": [
        "path_to_value_1",
        "path_to_value_2",
        "path_to_value_3"
    ]
}
```

And instruct the Scout2Listall tool to use this configuration using the `--keys-from-file` command line argument:

```
./Scout2Listall.py --profile <profile-name> --path ec2.regions.id.vpcs.id.security_groups.id --keys-from-file my-own-config.json
```

# 5. Arbitrary output

The `Scout2Listall` tool has a simple template engine that allows users create their own template in order to output Scout2 data to arbitrary formats. In the following example, we will output the list of IAM users ready for GitHub markdown, with the user names displayed in bold and italic.

```
$ Scout2Listall --config iam-user-without-mfa.json --format-file iam-users-without-mfa.md
```

The `iam-users-without-mfa.md` is defined as follow, and the output of the above command line has been included in this page just after.

```
This file is an example demonstrating how to use ```Scout2Listall``` to generate a
markdown file that contains the list of IAM user names matching a given Scout2
rule. It will create a list of users, whose name is bold and italic.

_ITEM_(* ***_KEY_(iam.users.id.name)***)_METI_
```

This file is an example demonstrating how to use ```Scout2Listall``` to generate a
markdown file that contains the list of IAM user names matching a given Scout2
rule. It will create a list of users, whose name is bold and italic.

* ***MisconfiguredUser-NoMFA1***
* ***MisconfiguredUser-NoMFA2***
* ***MisconfiguredUser-NoMFA3***

The `_ITEM_()_METI_` macro instructs the tool that the list of resources flagged by Scout2 should be rendered in this location. The value returned for each resource is defined between the parenthesis. In this example above, only the user's name is displayed. If we wanted to also output when the user was created, the _ITEM_ line may look as follow.

```
_ITEM_(* ***_KEY_(iam.users.id.name)***, created on _KEY_(iam.users.id.CreateDate))_METI_
```

Using this feature, users of Scout2 may leverage the Scout2Listall tool to output data into arbitrary formats.
