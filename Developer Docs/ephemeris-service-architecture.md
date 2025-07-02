# ephemeris-service Architecture

## Description

* `ephemeris-service-database`
    * Relational database.
    * Contains validated `Metadata` that is updated through the `ephemeris-service-api`.
* `ephemeris-service-api`
    * The API layer that performs C.R.U.D on the `ephemeris-service-database`.
* `ephemeris-service-web`
    * The web layer that provides a GUI for Fetch employees to interface with the `ephemeris-service-api`.
* `ephemeris-service-slack`
    * A slack bot that provides slack commands for Fetch employees to interface with the `ephemeris-service-api`.

## Diagram

![ephemeris-service-architecture](plantuml/ephemeris-service-architecture.png)

## Additional Architecture Considerations

* Multi-region deployments
    * Web, API, Lambda
    * Database
        * Regional read replicas with writes to the primary through transit gateway
* Single-region deployments
    * The primary (write) database
    * Slackbot
* Postgres vs DynamoDB
    * Chose a relational database so that we can make it easier to audit data changes.
* Concurrent Database Writes
    * Rare but possible if two people are submitting changes at the same time.
    * Slackbot can be made to post when
        * Database changes take place
        * Users access the web UI and select a service/pack for modification
        * A service/pack is modified multiple times in a short period of time
        * The post-modification diff is unexpected (may also be displayed on the web UI)