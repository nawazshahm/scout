# Introduction

Scout2 supports loading trusted CIDRs from a json file formatted in a fashion similar to [AWS' public IP ranges](http://docs.aws.amazon.com/general/latest/gr/aws-ip-ranges.html). At a high level, the file should is expected to look as follow.

```
{
    "createDate": "2017-12-02-07-08-36",
    "prefixes": [
        {
            "ip_prefix": "1.2.3.4",
            "name": "Location A"
        },
        {
            "ip_prefix": "5.6.7.8",
            "name": "Location B"
        }
    ]
}
```

# Step 1: Generate a custom CIDR list

The `awsrecipes_create_ip_ranges.py` [tool](https://github.com/nccgroup/AWS-recipes/blob/master/Python/awsrecipes_create_ip_ranges.py) may be used in order to generate well-formatted custom CIDR lists. The tool offers several use cases, such as generation from a CSV file, an interactive mode, and fetching data from AWS accounts to get names of EC2 instances and VPCs.

NCC Group published a blog post with additional information about usage of this tool. The blog post is available at https://www.nccgroup.trust/us/about-us/newsroom-and-events/blog/2015/november/efficient-review-of-aws-security-groups.

# Step 2: Provide the list of custom CIDRs to Scout2

By default, Scout2 displays the `name` attribute of each CIDR. Running the following results in the CIDR name to be displayed in parenthesis next to the CIDR in each of the security group grants.

```
$ Scout2 --ip-ranges ip-ranges-demo.json
```

In the event that you used an different attribute than name, for example, `office_name`. You may 

```
$ Scout2 --ip-ranges ip-ranges-demo.json --ip-ranges-name-key office_name
```

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-cidr-name-display-001.png)

The screenshot above illustrates that the name of the CIDR is displayed next to each security group's IP grant.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-cidr-name-display-002.png)

When an unknown CIDR is found, the `Unknown CIDR` caption is added to the report, which facilitates detection of EC2 security group rules that whitelist network traffic from untrusted IP ranges.


