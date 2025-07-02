# NORM

Norm is a Receipt Normalizer for ingesting receipts into Fetch.

## Structure

Example Project structure

* [https://github.com/golang-standards/project-layout](https://github.com/golang-standards/project-layout)
* [https://medium.com/@benbjohnson/structuring-applications-in-go-3b04be4ff091](https://medium.com/@benbjohnson/structuring-applications-in-go-3b04be4ff091)
* [https://youtu.be/1rxDzs0zgcE](https://youtu.be/1rxDzs0zgcE)
* [https://github.com/perkeep/perkeep](https://github.com/perkeep/perkeep)

# Running Locally

## Local Server

`cmd/server/main.go` will run a local Echo server which invokes Norm. This bypasses all the Lambda logic,
but invokes the norm App using `app/app.go` as the entry point.

JSON Receipts can be sent to `localhost:8000` as a POST request.

```
curl --location --request POST 'localhost:8000' \
--header 'Content-Type: application/json' \
--data-raw '{
  "eventType": "RECEIPT_SUBMITTED",
  "receipt":{
    "id": "8812b372-4916-4679-bf29-4e9653ed5588",
    "storeName": "walmart",
    "userId": "5cfad84ba87ed25ec1f418aa"
   }
}'
```

## Docker

Running by Docker uses an official AWS image that includes a lambda runtime environment.
This allows norm to be invoked with the AWS Lambda API.

Build the docker image:

```
docker build -t norm --build-arg GOPROXY=[GO Proxy to access JFrog] --build-arg GONOSUMDB=bitbucket.org/fetchrewards
```
Run the container

```
docker run -p 9000:8080 norm 
```

## Make

If you have `make` installed, you can run the following build targets:

| Target | Description | Notes |
|---|---|---|
| build-image | Builds the docker image with the name "norm" ||
| run | Runs the application as it would in lambda | This is probably not what you want--you probably want `run-local` |
| run-local | Runs the application as a local web service for local testing | |
| test | Runs all unit tests locally | |
| release | Builds the application artifact | |

### Reference

* [https://docs.aws.amazon.com/lambda/latest/dg/images-test.html](https://docs.aws.amazon.com/lambda/latest/dg/images-test.html)

NOTE: Documentation makes this look straight-forward. Currently showing errors suggesting REI isn't loaded as
advertised.

# Environment Setup

The `deployments` directory contains the CDK stack to deploy a Norm environment.

AWS's default CDK readme is in that directory for a CDK overview.

## Prerequisite

* AWS CDK installed on your machine. [Guide](https://docs.aws.amazon.com/cdk/latest/guide/home.html)
* AWS Credentials configured on your machine as for AWS CLI. [Docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)

## Editing the environment

The stack definition is in `deployments/lib/norm-infra-stack.ts`

Reference for how to configure resources can be found in the [CDK API Reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html)

## Deploying the Environment

Regardless of the deployment method used, the deployment must be defined as a stack in `/deployments/bin/`

Each region file has a place reserved where custom stacks can be defined.

### Manually

First build and upload a docker image of the code to deploy. Give the image a unique tag.

```
    - aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 292095839402.dkr.ecr.us-east-1.amazonaws.com
    - docker build -t 292095839402.dkr.ecr.us-east-1.amazonaws.com:[THE IMAGE TAG] . --build-arg GOPROXY=${GOPROXY} --build-arg GONOSUMDB=${GONOSUMDB}
    - docker push 292095839402.dkr.ecr.us-east-1.amazonaws.com:[THE IMAGE TAG]
```

After the image is published to the ECR Repo it can be deployed with the CDK.

```
cdk deploy [Your stack name] -r arn:aws:iam::292095839402:role/CloudFormation-Deploy --parameters codeTag=[The Image Tag]
```

Deployments are handled through the CDK. A stack can be defined in the desired region in `/deployments/bin/`

```
npx cdk deploy dev-us-east-1 --require-approval never -r arn:aws:iam::292095839402:role/CloudFormation-Deploy --parameters codeTag=$BITBUCKET_BRANCH
```

### Pipeline

To deploy your stack via pipeline. You can use the custom `deploy` pipeline. You'll be prompted for the stack name.

The pipeline will perform the manual deploy above with the specified stack. The image will be tagged with the branch name.