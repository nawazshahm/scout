

### Update the metadata

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