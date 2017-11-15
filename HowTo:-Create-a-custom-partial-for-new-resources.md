
After fetching additional resources, additional work in necessary in order to be able to visualize the resources in the HTML report: creation of a new AngularJS partial that corresponds to the new type of resources that have been fetched.

Any partial declared under ```AWSScout2/output/data/html/partials``` will be included in the generated HTML report; however, for clarity and consistency in the code, the naming convention for files that declare partials is as follow: ```services.<service_name>(.regions.id.)(.vpcs.id).<resource_name>.html```

This commit diff illustrates the creation of a new file that contains a partial for Lambda functions: https://github.com/nccgroup/Scout2/commit/2c8c385433a2bf7352e185030e0e02c585bff947#diff-c8b75a167a3bc6511835229cbba0ab05


```
    <!-- Lambda function partial -->
    <script id="services.awslambda.regions.id.functions.partial" type="text/x-handlebars-template">
        <div class="list-group-item active">
            <h4 class="list-group-item-heading">{{name}}</h4>
        </div>
        <div class="list-group-item">
            <h4 class="list-group-item-heading">Attributes</h4>
            {{> generic_object resource}}
        </div>
    </script>
    <script>
      Handlebars.registerPartial("services.awslambda.regions.id.functions", $("#services\\.awslambda\\.regions\\.id\\.functions\\.partial").html());
    </script>
```

In the snippet above, the first script element declares the Handlebars template that will be used for each resource (*i.e.* Lambda). In this case, no detailed work was performed and we used a generic partial that iterates through each attribute of the resource and displays its value. This is a convenient way to get started and may be particularly useful 
to get familiar with the object's attributes, but does not provide the best visualization experience (*e.g.* no pretty formatting, custom sections, and many attributes that are irrelevant from a security point of view are displayed).

The following snippet is the partial used to display SNS topics. It contains several sections (*i.e.* `list-group-item`) and uses an accordion element to enable collapsing/expanding the topic policy, which is displayed as JSON payload.

```
    <!-- SNS queue partial -->
    <script id="services.sqs.regions.id.queues.partial" type="text/x-handlebars-template">
        <div class="list-group-item active">
          <h4 class="list-group-item-heading">{{name}}</h4>
        </div>
        <div class="list-group-item">
          <h4 class="list-group-item-heading">Information</h4>
          <div class="list-group-item-text item-margin">Region: {{region}}</div>
          <div class="list-group-item-text item-margin">ARN: {{arn}}</div>
          <div class="list-group-item-text item-margin">Created on: {{CreatedTimestamp}}</div>
        </div>
        {{#if Policy.Statement.length}}
          <div class="list-group-item">
            {{> accordion_policy name = 'Access control policy' policy_path = (concat 'sqs.regions' region 'queues' @key 'Policy') document = Policy}}
          </div>
        {{/if}}
    </script>
    <script>
      Handlebars.registerPartial("services.sqs.regions.id.queues", $("#services\\.sqs\\.regions\\.id\\.queues\\.partial").html());
    </script>

    <!-- Single SNS queue template -->
    <script id="single_sqs_queue-template" type="text/x-handlebars-template">
        <div style="text-align: right; padding-right: 10px; text-weight: bold;"><a href="javascript:hidePopup()">X</a></div>
        {{> services.sqs.regions.id.queues}}
    </script>
    <script>
        var single_sqs_queue_template = Handlebars.compile($("#single_sqs_queue-template").html());
    </script>
```



