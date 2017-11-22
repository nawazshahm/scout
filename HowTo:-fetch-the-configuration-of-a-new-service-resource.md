# Introduction

This wiki page documents how the scope of Scout2 may be increased, to add support for either a new service or new resource type. Note that, if you plan on submitting a pull request to increase the scope of Scout2, the pull request should be justified by one of the following:

* Improvement of the quality of other findings (_e.g._ remove false positives in unused security group finding).
* New rules that apply to the new service / resources have been created.
* Visibility into this type of resource is important for security-related reviews.

## Step 1: Update The Metadata

1. Edit the file under `AWSScout2/configs/data/metadata.json`
1. Find the service group corresponding to the service
   1. If necessary, create a new service group by matching how AWS organizes services in the console.
1. Find the service corresponding to the new resource to be fetched
   1. If necessary, create a new service object in the service group with a `resources` attribute.
1. Add a new type of resource to be fetched at run time, specify the following:
   1. The boto3 API call that should be made (http://boto3.readthedocs.io/en/latest/reference/services/lambda.html)
   1. The name of the response's attribute that contains the list of resources (from the same boto3 documentation page)
   1. The path under which the resources will be stored: ```services.<service_name>(.regions.id)(.vpcs.id).<resource_name>```

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

## Step 2: Update The Services Configuration

If support for a new service was added, in addition to updating the metadata file, the ```ServicesConfig``` object must be updated such that it contains a new configuration variable for this service. The following commit illustrates the enabling support for AWS Lambda: https://github.com/nccgroup/Scout2/commit/ed001c6408aaadca5b9445accf50adef3816c0d5#diff-579dcd8f739afa19c5d767616494a9e3.

## Step 3: Create (or Update) The Service-Specific Configuration Class

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