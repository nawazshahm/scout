
## Step 1: Update the metadata

1. Edit the file under `AWSScout2/configs/data/metadata.json`
1. Find the service group corresponding to the new service
1. Add a new type of resource to be fetched at run time, specify the following:
   1. The boto3 API call that should be made (http://boto3.readthedocs.io/en/latest/reference/services/lambda.html)
   1. The name of the response's attribute that contains the list of resources (from the same boto3 documentation page)
   1. The path under which the resources will be stored: ```services.<service_name>.(regions.id).(vpcs.id).<resource_name>```

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

## Step 2: Update the Services configuration

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