![](https://github.com/nccgroup/Scout2/wiki/images/scout2-dashboard-head.png)

# What is Scout2?

Scout2 is a security tool that lets AWS administrators assess their environment's security posture. Using the AWS API, Scout2 gathers configuration data for manual inspection and highlights high-risk areas automatically. Rather than pouring through dozens of pages on the web, Scout2 supplies a clear view of the attack surface automatically.

Scout2 was designed by security consultants/auditor. It is meant to provide a point-in-time security-oriented view of the AWS account it was run in. Once the data has been gathered, all usage may be performed offline. 

For engineers in order to implement periodic and/or continuous review of their AWS environment, Scout2 may be used a base framework that provides. TODO TODO.

### Basic workflow

Assuming access to the AWS APIs has already been configured on a machine (_e.g._ you can use the AWS CLI), then installing and using Scout2 should be trivial:

1. Install Scout2
```
pip install awsscout2
```

2. Run the tool
```
Scout2 (--profile <profile-name>)
```

3. Browse the HTML report that is automatically open in the default web browser

### Advanced usage

1. Generate a list of trusted IP ranges
2. Generate a custom ruleset
3. Provide Scout2 with the custom ruleset and trusted IP ranges
