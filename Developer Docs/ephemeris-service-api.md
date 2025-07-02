# ephemeris-service API

Ephemeris is used to tag services and packs with metadata. Here are a few basic instructions on how to operate our Ephemeris Fetch inventory system through basic CREATE, READ, UPDATE, DELETE instructions:

---

## CREATE A SERVICE

````
% curl -X POST "https://dev-ephemeris-service-api.fetchrewards.com/service/test-marcelo-service"
````

---

## UPDATE A SERVICE (ADDING METADATA)

```
% curl -X PUT "https://dev-ephemeris-service-api.fetchrewards.com/service/test-marcelo-service/meta"  -H "Content-Type: application/json" --data-raw '{"key": "pack", "value": "sre"}'
{"message": "Updated test-marcelo-service/pack = sre"}
```

---

## READ SERVICE

```
% curl -X GET "https://dev-ephemeris-service-api.fetchrewards.com/service/test-marcelo-service"
{
    "last_modified_time_utc": "2022-11-10",
    "service_name": "test-marcelo-service"
}
```
Not very interesting as it doesn’t have any data. Czech out the next one:

---

## READ METADATA ASSOCIATED WITH A SERVICE

```
% curl -X GET "https://dev-ephemeris-service-api.fetchrewards.com/service/test-marcelo-service/meta"
[
    {
        "service_name": "test-marcelo-service",
        "last_modified_time_utc": "2022-11-10",
        "field_value": "sre",
        "field_name": "pack",
        "region": "*",
        "environment": "*"
    }
]
```
This response payload will return a json block for each key/value piece of metadata.

---

## DELETE METADATA FROM A SERVICE

**BE CAREFUL**

```
% curl -X DELETE "https://dev-ephemeris-service-api.fetchrewards.com/service/test-marcelo-service/meta"  -H "Content-Type: application/json" --data-raw '{"key": "pack", "value": "sre"}'
{"message": "Deleted: test-marcelo-service/pack"}
```

---

## DELETING A SERVICE

Currently not supported

---

## Notes

* We need to be able to perform C.R.U.D against the `ephemeris-service-database` through an API.
* Our deployment systems will be requesting tags/labels for AWS/Kubernetes resources.
  * We will only send `Metadata` key/value pairs to be used as tags/labels if they conform to the applicable AWS or Kubernetes syntax requirements.
* We should allow querying based on substring as well as full string, so we can acquire metadata with queries like "all metadata for service-x that contains 'wiki'"

---

## Deployment Pipeline Access

* We can use a VPC Lambda as a go-between for Bitbucket pipelines
    * Our Bitbucket pipelines already have access to the VPC through AWS credentials
    * We would remove the need to expose ephemeris-service-api as a public website
    * Internal Fetch software could still access the API through the URLs for the API and wouldn't need to go through the Lambda
* service-deploy will pull tags using an in-house ansible module
    * If we use the Lambda go-between, the ansible module would simply invoke the lambda function to acquire the tags it needs
    * The lambda will be provided all the data (service, region, environment) necessary to acquire metadata for the service, including a filtering mode parameter that will make sure any metadata returned will meet a certain criteria (such as AWS tag requirements).
    * Another alternative implementation we can consider would be to push metadata to service-deploy in the form of a single YAML file that gets updated periodically or on-change-to-ephemeris-service-metadata. Then a service would just key into the data defined in that file to get its tags without needing to perform a lambda invocation.
* cloud-deploy can work similarly to service-deploy
    * Perhaps we will use https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/lambda_invocation
* kubernetes (apollo)
    * Using our kubernetes bitbucket pipe to update the helm charts with appropriate tagging
    * Could call the same Lambda being used by service-deploy/cloud-deploy, or perhaps make the HTTP request directly to ephemeris-service

### Comprehensive Diagram

![ephemeris-service-deploy-sequence](plantuml/ephemeris-service-deploy-sequence.png)

### Service Deploy Sequence

![ephemeris-service-service-deploy-sequence](plantuml/ephemeris-service-service-deploy-sequence.png)

### Terraform Sequence

![ephemeris-service-terraform-sequence](plantuml/ephemeris-service-terraform-sequence.png)

### Kubernetes Sequence

![ephemeris-service-kubernetes-sequence](plantuml/ephemeris-service-kubernetes-sequence.png)