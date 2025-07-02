# Offer-service Deployment

Feature development: Due to the tumultuous nature of non-prod environments
pull-requests may be deployed to any of `dev`, `stage`, or `preprod`.
See pull-request Pipeline for details.

Master: Changes-sets merged into master synchronously trigger a release, a `preprod` deploy, then a `prod` deploy.

Terraform: Info about how offer-service uses terraform is [here](https://fetchrewards.atlassian.net/wiki/spaces/OP/pages/3414720530/Terraform)

### How-To

`dev|stage|preprod` Recommended:

* In a pull-request pipeline, click the `Run` button to-the-right of the desired environment

`dev|stage|preprod` Custom:

* Alternatively, navigate to [Pipelines](https://bitbucket.org/fetchrewards/offer-service/addon/pipelines/home)
* Select `Run Pipeline` in the top-right
* Select the desired branch
* Select `custom: <desired-env>` from the Pipeline dropdown

### Prod

* Notify **_both_** `#pack-moneyball` and `#collab-deploy-coordination` of forthcoming deploy
* Merge changes into the master branch
* Navigate to the [offer-service dashboard](https://fetchrewards.grafana.net/d/asxqDGTMk/offer-service?orgId=1&refresh=30s) and begin monitoring telemetry
* Deploy to `preprod`: Click `Run` to-the-right of `Deploy to Preprod`
* Begin `prod` deploy: Click `Run` to-the-right of `Deploy to Prod`
* Navigate to the `prod` child-pipeline (link is listed in `pipe: atlassian/trigger-pipeline` logs)
* Once `Create Environment` is complete and **_BEFORE_** running `Swap Environment` perform the following:
* Provision ALB capacity via [this guide](https://fetchrewards.atlassian.net/wiki/spaces/DEVOPS/pages/2842132481/AWS+Elastic+Load+Balancer+ALB+NLB+ELB)
* TBD: vegeta script
* Swap the ElasticBeanstalk CNAMEs: Click `Run` to-the-right of `Swap Environment`
* Ensure message is published in `#app-prod-deploy-notifications`
* Terminate old environment:
  * Wait at least until next business day
  * Ensure old environment is no longer receiving traffic (request sum should be zero)
  * Click `Run` to-the-right of `Terminate Prod`
* Remove provisioned capacity added via [this guide](https://fetchrewards.atlassian.net/wiki/spaces/DEVOPS/pages/2842132481/AWS+Elastic+Load+Balancer+ALB+NLB+ELB)

### Details

`dev|stage|preprod` Custom/Triggered Deploy Pipeline(s):

* Build an artifact (on the specified branch)
* Upload artifact to ElasticBeanstalk
* Create a new ElasticBeanstalk environment with `service-deploy`
* Swap the ElasticBeanstalk CNAMEs with `deployer`
* (`preprod` only) Trigger beholder tests with `deployer`

`prod` Custom/Triggered Deploy Pipeline:

* Build an artifact (on the specified branch)
* Upload artifact to ElasticBeanstalk
* Create a new ElasticBeanstalk environment with `service-deploy`
* (Manual) Swap the ElasticBeanstalk CNAMEs with `deployer`
* Notify `#app-prod-deploy-notifications`
* Wait for 60 minutes for old-env traffic to drain
* (Manual) Terminate old-environment
