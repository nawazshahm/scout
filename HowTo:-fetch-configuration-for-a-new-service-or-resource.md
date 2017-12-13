# Introduction

This wiki page documents how the scope of Scout2 may be increased, to add support for either a new service or new resource type. Note that, if you plan on submitting a pull request to increase the scope of Scout2, the pull request should be justified by one of the following:

* Improvement of the quality of other findings (_e.g._ remove false positives in unused security group finding).
* New rules that apply to the new service / resources have been created.
* Visibility into this type of resource is important for security-related reviews.

## Step 1: Update the metadata

1. Edit the file under `AWSScout2/configs/data/metadata.json`
1. Find the service group corresponding to the service
   1. If necessary, create a new service group by matching how AWS organizes services in the console.
1. Find the service corresponding to the new resource to be fetched
   1. If necessary, create a new service object in the service group with a `resources` attribute.
1. Add a new type of resource to be fetched at run time, specify the following:
   1. The boto3 API call that should be made (http://boto3.readthedocs.io/en/latest/reference/services/lambda.html)
   1. The name of the response's attribute that contains the list of resources (from the same boto3 documentation page)
   1. The path under which the resources will be stored: ```services.<service_name>(.regions.id)(.vpcs.id).<resource_name>```

**Note that the name of the resources in the Scout2 data structure should be in snake case and plural, *i.e.* functions, db_subnet, resource_arns, ...**

In this example, the following payload was added in the ```compute``` service group, as can be seen at https://github.com/nccgroup/Scout2/commit/ed001c6408aaadca5b9445accf50adef3816c0d5#diff-d3ca4ab2899d83e6eb513318af837b8d.
```
        "awslambda": {
            "resources": {
                "functions": {
                    "api_call": "list_functions",
                    "response": "Functions",
                    "path": "services.awslambda.regions.id.functions"
                }
            }
        }
```

This metadata file is used at runtime for the following tasks:
* Determine the list of all resources that must be fetched.
* Update the HTML report navigatiob bar.

***Note that changes to this files are not reflected after a ```--local``` run.***

## Step 2: Update the services configuration

If support for a new service was added, in addition to updating the metadata file, the ```ServicesConfig``` object must be updated such that it contains a new configuration variable for this service. The following commit illustrates the enabling support for AWS Lambda: https://github.com/nccgroup/Scout2/commit/ed001c6408aaadca5b9445accf50adef3816c0d5#diff-579dcd8f739afa19c5d767616494a9e3.

## Step 3: Create (or update) the service-specific configuration class

The following snippet illustrates the most basic service-specific class that may be created. It declares two classes:

* *LambdaRegionConfig*, used to store the service's configuration within a single region.
* *LambdaConfig*, which stores one *LambdaRegionConfig* object per region in which the service is enabled.

```
# -*- coding: utf-8 -*-
"""
Lambda-related classes and functions
"""

from AWSScout2.configs.regions import RegionalServiceConfig, RegionConfig



########################################
# LambdaRegionConfig
########################################

class LambdaRegionConfig(RegionConfig):

    pass



########################################
# LambdaConfig
########################################

class LambdaConfig(RegionalServiceConfig):
    """
    Lambda configuration for all AWS regions
    """

    region_config_class = LambdaRegionConfig

    def __init__(self, service_metadata, thread_config = 4):
        super(LambdaConfig, self).__init__(service_metadata, thread_config)
```

https://github.com/nccgroup/Scout2/commit/ed001c6408aaadca5b9445accf50adef3816c0d5#diff-5e845193109c8c0bc5e2763cdb6816d1

## Step 3bis: Test that your fetching code works

**Note that this step is only necessary for testing or if custom parsing of the resource is not necessary.**

At this point, running Scout2 would result in the API calls to be made, resources to be passed to a generic `store_target` function; however, this would fail due to Scout2 not knowing which resource attribute should be used as the resource identifier. This can be specified in `AWSScout2/configs/__init__.py` by adding an entry in the `resource_id_map` dictionary. The key should be matching the snake_case name for the new resources being fetched, and the value should be the name of the attribute to be used as the identifier.

```
resource_id_map = {
    'resource_arns': 'ResourceARN',
    'network_interfaces': 'NetworkInterfaceId',
    'peering_connections': 'VpcPeeringConnectionId',
    'subnet_groups': 'DBSubnetGroupName'
}
```

**Note that the Scout2 UI will likely have bugs if AWS enables dots (.) in values of the resource identifier attribute. In such case, refer to Step 5.**

## Step 4: Create resource-specific parsing functions

In the example above, the entire resource object is stored in the AWS configuration file used by Scout2, which is overkill as in many cases only several attributes are relevant to perform a security review. In other cases, the data fetched may need to be formatted in order to work better with existing code, or completed by making another API call.

For example, when support for SNS topics was created, a custom parser had to be created due to the fact that topic attributes aren't returned when making `sns:ListTopics` API calls. For each topic, an additional API call to the `sns:GetTopicAttributes` endpoint is necessary. This resulted in the following custom parser for SNS topics:

```
    def parse_topic(self, params, region, topic):
        """
        Parse a single topic and fetch additional attributes

        :param params:                  Global parameters (defaults to {})
        :param topic:                   SNS Topic
        """
        topic['arn'] = topic.pop('TopicArn')
        topic['name'] = topic['arn'].split(':')[-1]
        (prefix, partition, service, region, account, name) = topic['arn'].split(':')
        api_client = api_clients[region]
        attributes = api_client.get_topic_attributes(TopicArn=topic['arn'])['Attributes']
        for k in ['Owner', 'DisplayName']:
            topic[k] = attributes[k] if k in attributes else None
        for k in ['Policy', 'DeliveryPolicy', 'EffectiveDeliveryPolicy']:
            topic[k] = json.loads(attributes[k]) if k in attributes else None
        topic['name'] = topic['arn'].split(':')[-1]
        manage_dictionary(topic, 'subscriptions', {})
        manage_dictionary(topic, 'subscriptions_count', 0)
        self.topics[topic['name']] = topic
```

The name of the parsing function has to be `parse_<resource_type>`, and the input will always be as follow:
* `params`: Global parameters, unnecessary in most situations.
* `region`: Region in which the resource is definedl
* `topic`: Single resource as returned by the `list` or `describe` API call.

Note that, regardless of the resource's attributes, the default for Scout2 is that the resource's ARN is accessible via the `arn` attribute, and that its name or identifier is accessible via the `name` attribute. This is why, in this case, the `TopicArn` attribute is removed and used to populate the value of the `arn` and `name` attributes.

Subsequently, calls to the `GetTopicAttributes` API are made, and the results are stored within the `topic` object. Additional placeholders for the topic subscriptions are created.

Finally, the `topic` object is saved within the dictionary of topics of the instance of the `SNSRegionConfig` class.

## Step 5 (optional): Create a custom identifier value.

If the value of the resource identifier can contain a dot (.), this is likely to cause UI bugs in the Scout2 HTML report. In this case, the `get_non_aws_id` function may be used. The following code snippet illustrates how a custom parsing function may look like.

```
resource_id = self.get_non_aws_id(resource[name])
self.resources[resource_id] = resource
```
