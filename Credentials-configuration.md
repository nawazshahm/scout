**TL;DR:**

  1. **If you're able to use the AWS CLI, Scout2 should "just run".**
  2. **If you use profile names, just use the `--profile` argument when running Scout2.**

# Introduction

Scout2 was designed to work seamlessly on machines used to make AWS API calls, which includes

1. Developer machines configured to use the AWS CLI or any other tool based on AWS official SDKs.
2. EC2 instances

In the following section, we will discuss in further details various configurations 

# Option: AWS credentials file

If you've used the AWS CLI or any tool based on one of the AWS SDKs, chances are you have configured your environment such that credentials are ready to be used. This could be the result from running the `aws configure` command, for example. In practice, your AWS credentials are stored in a file under `~/.aws/credentials`.

```
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar
```

This means that, if you run a command such as `aws iam list-users`, the credentials will be read from the `default` profile. For users who interact with multiple AWS accounts, AWS allows to have profiles. This would result in a configuration file looking as follow.

```
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar
[profile1]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar1
[profile2]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar2
```

In this configuration, users may now run the AWS CLI and choose the pair of credentials with the `--profile <profilename>` argument, where profile name would be `profile1` or `profile2`. Similar to the AWS CLI, Scout2 supports profiles and could be run using the credentials associated with any of these profiles.

# Option: AWS credentials file with MFA

If MFA-protected API access has been enabled in your account, use of the access key ID and secret key may be limited to several actions until STS credentials have been obtained. To facilitate generation of STS credentials, Scout2 supports reading the MFA device serial number from the AWS credentials file. The MFA device serial (ARN) may be specified using the `mfa_serial` or `aws_mfa_serial` keyword, as illustrated below.

```
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar
[profile1]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar1
aws_mfa_serial = arn:...
[profile2]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar2
mfa_serial = arn:...
```

In this case, Scout2 will automatically prompt for the MFA code at runtime if generation of new STS credentials is necessary.

```
$ Scout2.py --profile profile1
Saved STS credentials expired on 2017-11-06 01:13:14+00:00
Enter your MFA code (or 'q' to abort): 
 123456
Connecting to AWS sts...
Fetching S3 config...
```

The underlying library, opinel, would actually modify the credentials file as follow: the long-lived AKIA access key and its associated profile will be renamed with the original profile name appended by `-nomfa`, and the STS short lived credentials stored as `profile1`.

```
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar
[profile1]
aws_access_key_id = ASIA...
aws_secret_access_key = foobartmp1
aws_session_token = tmpsessiontoken
expiration = some_time
[profile2]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar2
mfa_serial = arn:...
[profile1-nomfa]
aws_access_key_id = AKIA...
aws_secret_access_key = foobar1
aws_mfa_serial = arn:...
```

# Option: AWS credentials file with roles

A relatively common way to access the AWS APIs now is to assume a particular role, receive a new set of STS credentials, and use these for subsequent API calls. Scout2 supports this functionality as well. The easiest way to use IAM roles with Scout2 (and the AWS CLI) is to configure a new profile in the `~/.aws/config` file, as illustrated below.

```
[profile profile1-demorole]
role_arn = arn:aws:iam::123456789012:role/Demo
source_profile = profile1
[profile profile2-demorole]
role_arn = arn:aws:iam::0987654321098:role/Demo
source_profile = profile2
```

In this example, two IAM roles have been configured and associated with a profile for which credentials have already been configured in the `~/.aws/credentials` file. Using Scout2 (or the AWS CLI) with the `--profile profile1-demorole` will result in an API call to `sts:assumerole` be made, short-lived credentials be stored under the `~/.aws/cli/cache` folder, and the tool to run in the role's context.

In the event of profile1-demorole and profile1 requires MFA, Scout2 will prompt the user for their MFA code. In case the source profile does not specify an MFA device, the call to `sts:assumerole` will happen automatically. Note that role credentials are only valid for one hour. As a result, Scout2 will re-assume the role if the cached credentials have expired.

# Option: environment variables

If credentials have been configured as environment variables, Scout2 will use these when making API calls.

```
$ export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
$ export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

# Option: from an EC2 instance with an IAM role

When run from an EC2 instance with an IAM role, Scout2 will use the role's credentials (read at the metadata URI) unless other credentials have been provided.

```
$ Scout2
```

# Option: CSV file

If an IAM user downloaded their AWS API key in a CSV file, Scout2 may read these directly from the file using the `--csv-credentials` argument.

```
$ Scout2 --csv-credentials <CREDENTIALS.CSV>
```

