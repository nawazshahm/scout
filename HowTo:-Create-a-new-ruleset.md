# Introduction

Scout2 provides a default set of rules that the authors have chosen to be the best compromise between breadth of coverage and a minimum number of false positives. While this default ruleset provides many insights into the configuration of AWS environments, using a custom ruleset will provide significantly improved value. We strongly recommend frequent users of the tool generate their own set of rules.

# Scout2RulesetGenerator

Shipped along with Scout2, the Scout2RulesetGenerator tool provides a simple, graphic interface to generate and maintain custom rulesets. If starting from scratch, use the following command line to start the tool:


```
$ Scout2RulesGenerator --ruleset-name myruleset
```

This will generate and open an HTML page in the browser, which can then be used to create a new ruleset whose name will be `myruleset`. The HTML Ruleset Generator tool is organized in a similar way that Scout2's report, with the navigation bar enabling display of rules within a given service.












