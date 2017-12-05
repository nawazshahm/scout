# Introduction

In addition to supporting custom rules and custom rulesets, Scout2 enables users to flag certain resources as exceptions. For example, you may want to mark the S3 bucket that receives S3 access logs as an exception to the rule flagging S3 buckets for which access logs has not been enabled. This wiki page illustrates how one may generate a list of exceptions and use it when running the Scout2 analysis.

# Step 1: Creation of a list of exceptions

The Scout2 HTML report is the UI that may be used to create a list of exceptions. The following screenshot is from the IAM dashboard in a test account. In this example, we will mark the IAM group that contains an inline policy as an exception (center rule on the 2nd row in the screenshot).

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-exceptions-001.png)

The first step is to click on this dashboard element to display the list of resources flagged by the rule. In this case, only one IAM group is not compliant. After clicking on the IAM group inline policy details, we see that one statement is a combination of "Allow" and "NotAction", as illustrated below.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-exceptions-002.png)

Clicking on the DOM element highlighted in red will cause a JavaScript box to be displayed, asking whether this resource should be added to the list of exceptions for this particular rule.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-exceptions-003.png)

Clicking on the "OK" button will update the list of exceptions; however, the Scout2 results have not been updated at this time. In order to take the list of exceptions in account, you must click on the "Help" drop down menu and select the "Export exceptions" option.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-exceptions-004.png)

This will cause a file download dialog to show up, in which you may rename the file if wanted and save it on disk.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-exceptions-005.png)

# Step 2: Run Scout2 with a list of exceptions

In order to update the Scout2 report, you will need to re-run Scout2. Because all the configuration has been fetched already, there is no need to re-do all the API calls. A local run may be performed with the following command:

```
$ Scout2 --profile <profile-name> --local --exceptions ~/Downloads/myexceptions.json --no-browser
```

The `--no-browser` option means that Scout2 will not open the report in a new browser window, this is optional. If you choose to do so, you then need to refresh the Scout2 report in your browser. After clicking on the link to view the IAM dashboard, you will be able to confirm that the list of exceptions has been taken in account.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-exceptions-006.png)

Note that, from here, if you choose to generate a new list of exception. The list of exceptions that you will generate will include the exceptions used during the last run, as well as the exceptions just set in the UI.