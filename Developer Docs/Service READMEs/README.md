# receipt-detail-service

receipt-detail-service is a tailored service to provide data to the receipt-detail page of the mobile app.

receipt-detail-service is responsible for:
- Serving receipt-detail objects via REST endpoint for a specific Receipt ID
- Emitting receipt events via Web Socket so Mobile devices do not need to poll the back end after scanning a receipt

![image](./docs/receipt-detail-service.png)

**Note**: This is a rough diagram of the initial conceptual design of the service. It’s not intended to be prescriptive of the final implementation. This can also be updated as the service is implemented.

## Getting Started

First, install [Serverless Framework](https://www.serverless.com/framework/docs/getting-started) and Make.  

The following command will spin up the docker compose cluster with the `receipt-detail-service` along with a Redis cache.

```sh
make local
```  

Once the cluster is up, you can run the following command to invoke the lambda locally that updates the data model in Redis:  

```sh
make EVENT_FILE=${event_file} invoke
```  

It requires an event file that contains an SQS Event. Example JSON files can be found in `sample/`.

## Deployment 

The serverless components can be deployed with the following command:  

```sh
make STAGE=${stage} deploy-serverless
```  

The ECS Service can be deployed using `service-deploy`

## Support

**Slack**: [#pack-aperture](https://fetchrewards.slack.com/archives/C01RQJ73341)