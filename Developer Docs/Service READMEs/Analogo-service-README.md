# Analogo

Analogo (uh-nal-o-go) serves as a drop-in replacement for the duplicate receipt checking happening in RCS and FRS. It abstracts the duplicate detection process behind a service and removes the dependency on Mongo.

## How does it work?

---

Analogo hashes some details on receipts and stores them in a cache for fast retrieval. In the case of a duplicate hash (hash collision), potential matches of a submitted receipt are retrieved from a data store and interrogated more intricately. If those match according to some guidelines, the submitted receipt will be considered a duplicate.

See the following [diagram](https://lucid.app/documents/embeddedchart/11b72fb5-67a8-4b08-84df-d32a1c15d7c7) for more details.

### What's a "shim"?

**Shims** in Analogo refer to small pieces of logic that alter the consistency of duplicate detection - typically, that means excluding some small subset of all receipts from duplicate checks. Shims are intended to be *temporary* solutions, primarily for false duplicates. Behavior that needs to be changed long term should have a planned, external solution from Analogo before being integrated.

When a shim evaluates to `true` for a receipt `r`, `/api/duplicates/r` will return 0 duplicate ID's and `/api/duplicates/is` will always return a `false` DTO. 

### [Ignored Retailers](https://bitbucket.org/fetchrewards/analogo/src/main/internal/duplicate/shim/for_store_names.go)

This shim ignores a set of store names such that a receipt with a matching retailer will always be considered unique. To edit the list of store names, find the `analogo/ignored-retailers` flag in [Flagr](https://prod-flagr.fetchrewards.com/#/flags/21), then add or delete a segment whose description matches the corresponding retailer's name (the name is matched case-insensitive). In roughly 15 seconds, the service will refresh and include this store name in the shim.

## Getting Started

---

Simply execute `make` - this will first spin up the Docker Compose services, which holds all the dependencies Analogo needs to run locally, then execute the Analogo service with local arguments.

**Tip**: To later terminate the services running in Docker, execute `make local-env-down`.

## Testing

---

To test the application, run

```
make test
```

**Note**: By default, this will give tests a maximum timeout of `1s`. If you wish to specify a longer timeout (`-timeout=...`), you may, but **if you provide a timeout greater than 1m, the default timeout is used instead.** This is to avoid erroneously using Go's default test timeouts that can be upwards of 10 minutes.

To run integrations tests, you can use the Make target:

```
make test-integrations
```

If you wish to run the tests by hand, you'll need to specify the `it` or `integration` build flag:

```
go test -tags it ./...
```

## What is this `-ldflags` thing I see in the build?

---

`-ldflags` is an option of `go build` that allows for specifying linker options. Analogo utilizes `-ldflags "-X ..."` for settings some values at compile-time (i.e. the application version and Git hash).

 For more information, see a detailed explanation of Go's linker options [here](https://pkg.go.dev/cmd/link).

| Build Argument    | Expected Value                 | Purpose                                                         |
|-------------------|--------------------------------|-----------------------------------------------------------------|
| `VERSION`         | Current Git tag or commit hash | Rather self-explanatory; the version of the application.        |
| `BUILD_TIMESTAMP` | Unix timestamp (seconds)       | The Unix timestamp representing when the application was built. |

## Additional Resources

----

Want to learn more? Check out these resources:

- [Grafana](https://fetchrewards.grafana.net/d/49JuSZc7z/analogo?orgId=1)

- [System Diagrams](https://lucid.app/lucidchart/11b72fb5-67a8-4b08-84df-d32a1c15d7c7/edit?viewport_loc=60%2C21%2C2219%2C1104%2CIHoqPqLhr2gW&invitationId=inv_f26b62f3-6ac4-46ca-a72d-9e29b2c9047c)

## Support

----

Need some extra support? Contact one of the people or groups listed here:

**Packs:** Pack Aperture

**Slack Channels:** [#err-analogo](https://fetchrewards.slack.com/archives/C02HP2BB342)