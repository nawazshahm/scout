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

In case the rule description is not clear enough, it is possible to inspect the JSON payload that is used to define the rule by clicking on the `Details` button next to the Raw JSON section. The following section contains the definition of the IAM rule that flags IAM users for which an MFA device has not been setup. You can see that two conditions are required to flag an entity:

1. The user must have login profile (_i.e._ can access the web console via a username + password)
2. The list of MFA devices associated with the user is empty

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-004.png)

# Step 2: Modify rules as needed

### Enable / Disable rules

As mentioned above, a simple click in the checkbox for each rule allows enabling / disabling 

### Modify the level associated with the rule

For each rule, the level (_i.e._ warning or danger) may be set in the ruleset by using the dropdown options on the right handside as illustrated in the following screenshot.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-003.png)

### Configure parameterized rules

Some rules require parameters, which provides the following advantages:

1. A rule definition may be referenced multiple times in the ruleset, with only its arguments' values changing.
2. Rules that require environment-specific values, such as IP addresses or security group IDs are defined identically for any Scout2 user.

The screenshot below illustrates how a parameterized rule typically looks like in the Scout2RulesetGenerator. For documentation purposes, the Raw JSON details are presented; however, there is no need to review these when creating a new ruleset. For each parameter, a input field is available along with a short description of what the parameter is.

In this example, the rule takes two arguments:

1. The friendly/display name for the type of instances; in this case, "beefy".
2. The list of EC2 instance types considered as "beefy", each value separated by a comma.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-005.png)

1. Green boxes: References to the parameters are specified using \_ARG\_N\_ where N in the argument index, starting at 0.
2. Purple box: Explanation of what each argument is, to be displayed in the ruleset generator UI.
3. Blue boxes: Rendering of the explanation of what each argument is.
4. Red boxes: Value of each argument
5. Yellow boxes: Rendering of the argument 0's value in the ruleset generator UI.

_Note_: Refer to the [Create a new rule](https://github.com/nccgroup/Scout2/wiki/HowTo:-Create-a-new-rule) wiki page for further details about the JSON definition of Scout2 rules.

#### Rules with trusted CIDRs

Rules that use a list of known CIDRs are parameterized; however, the parameter value is passed from the CLI argument --ip-ranges. The following screenshot shows how the rule "Security Group Whitelists Unknown CIDRs" rule is defined.

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-006.png)

Refer to the [Use trusted CIDRs](https://github.com/nccgroup/Scout2/wiki/HowTo:-Use-a-list-of-trusted-CIDRs) wiki page for further details. 

# Step 3: Generate the ruleset

Click on the top right "Generate Ruleset" link, download and save the file named `myruleset.json`. 

![](https://github.com/nccgroup/Scout2/wiki/images/scout2-rulesetgenerator-007.png)
