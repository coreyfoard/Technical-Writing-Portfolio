# Receipt Service #

Receipt Service is the persistence layer for receipt data. It’s responsible for saving and retrieving receipt data. The method of storage is considered an implementation detail and out of scope when defining the API.

## Running Locally

## Documentation

Documentation for this service's provided API is published directly alongside the service using echo-swagger at:
`/swagger/index.html`.  The documentation is created using some instrumented comments,
so make sure to add them when creating new handler functions.

### Generating Documentation

To generate documentation using [swaggo/swag](https://github.com/swaggo/swag) install swag:
```text
go get github.com/swaggo/swag/cmd/swag@v1.6.6
```

And issue the following command to generate the docs for the whole project:
```text
swag init
```

On Linux you may need to add `$HOME/go/bin` to your `$PATH` to find the swag binary.

### Configure jFrog Artifcatory

If you haven't already, make sure you setup your `GOPROXY` and other Go environment variables.  Documentation can 
be found [here](https://fetchrewards.atlassian.net/wiki/spaces/DEV/pages/edit-v2/1252818979)

### Environment Selection ###

The configuration loaded is dependent on an environment variable called `ENVIRONMENT`. The value is directly correlated
to which yml config file is loaded from `./config/*.yml`. For example, setting the environment variable like 
`ENVIRONMENT=dev` will load the `dev.yml` config file.

* From command line `go run main.go`


### Docker ###

Info on using Docker to run this service

## Testing ##

### From the command line

`go test ./...`

If you want verbose output:

`go test ./... -v`

### In GoLand

You can right click to run individual tests/test files from the context menu.

### Integration Tests

Integration Tests can be found in the `/test` directory. And are marked with the `integration` build flag.

```
// +build integration
```

These tests are ran against live environments so are not CI/CD-safe and should only be ran manually.

To run these tests `cd` into the test directory and run `go test --tags=integration`

### Integration Tests

Integration Tests can be found in the `/test` directory. And are marked with the `integration` build flag.

```
// +build integration
```

Currently integration tests only run locally, but you can execute them using something like the following command:
```text
ENVIRONMENT=preprod go test -tags integration ./...
```

## Deployment ##

How do I deploy this thing?

## Config ##

We are making use of [Viper](https://github.com/spf13/viper) to manage our configuration.
