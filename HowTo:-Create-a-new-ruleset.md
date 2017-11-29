# Introduction

Scout2 provides a default set of rules that the authors have chosen to be the best compromise between breadth of coverage and a minimum number of false positives. While this default ruleset provides many insights into the configuration of AWS environments, using a custom ruleset will provide significantly improved value. We strongly recommend frequent users of the tool generate their own set of rules.

# Step 1: Open the Scout2RulesetGenerator

Shipped along with Scout2, the Scout2RulesetGenerator tool provides a simple, graphic interface to generate and maintain custom rulesets. If starting from scratch, use the following command line to start the tool:


```
$ Scout2RulesGenerator --ruleset-name myruleset
```

This will generate and open an HTML page in the browser, which can then be used to create a new ruleset whose name will be `myruleset`. The HTML Ruleset Generator tool is organized in a similar way that Scout2's report, with the navigation bar enabling display of rules within a given service.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-001.png)

The ruleset loaded will reflect the configuration of the default ruleset for Scout2. Any built-in rule that was not enabled and/or referenced in the default ruleset will appear in this tool, in a disabled state. The screenshot below illustrate this behavior.

The rule flagging "default security groups without an empty set of rule" is enabled, while the rule flagging use of a specific type of instance (t2.micro in this example) is not enabled.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-002.png)

For each rule, the level (_i.e._ warning or danger) may be set in the ruleset, by using the dropdown options on the right handside as illustrated in the following screenshot.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-003.png)

In case the rule description is not clear enough, it is possible to inspect the JSON payload that is used to define the rule by clicking on the `Details` button next to the Raw JSON section. The following section contains the definition of the IAM rule that flags IAM users for which an MFA device has not been setup. You can see that two conditions are required to flag an entity:

1. The user must have login profile (_i.e._ can access the web console via a username + password)
2. The list of MFA devices associated with the user is empty

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-004.png)

