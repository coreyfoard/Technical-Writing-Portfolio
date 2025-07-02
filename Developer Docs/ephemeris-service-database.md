# ephemeris-service Database

Initially we will have two types of tables for both packs and services in the database.
* A primary table, containing rarely changing fields or fields that will always be present.
* A `_meta` table, containing more frequently changing fields or user-defined fields.

 Notes for `region` and `environment`:
* We can use a overriding system where `*` (or `NULL` field value) for `region`/`environment` indicates that the row contains the default value for a particular key, but any row with a specific `region` or `environment` value (like `prod`) with the same key will override that default.
* This way, in the initial implementation we can ignore `region`/`environment` by just using something like `*` for all records until we have a reason to do an override.
* This also means we'll want to provide `region` and `environment` on the input to the API, just so we already have them for when we do want to make use of them.
* Implementing these fields could be considered premature, since we don't currently have a good example of why you would want to differentiate this data by region and environment, but because it's simple enough to perform the distinction up front (just a couple fields with default `*` or `NULL`) we don't currently think the cost of accounting for it is that high.

Notes for `last_modified_time_utc`:
* The idea of this field is that we could use it to let users know that a modification just took place on the pack or service they're about to update.
    * Similarly, on submission of new or updated data, the API could check to see if an update occurred on the same pack/service within the last, say, 10 seconds, and alert the user of this.
* It could be argued that this field doesn't even serve enough of a valuable purpose that we should bother with it.
    * We could do a before-and-after-expectation `diff` instead of checking for recent timestamps, which could be more accurate and reliable.
* Similarly to the `region` and `environment` fields, though, it doesn't really hurt us to maintain this timestamp if it provides us value.

## Pack Table

| pack_name                          | last_modified_time_utc                                       |
| ---------------------------------- | ------------------------------------------------------------ |
| The name of the pack. Primary key. | The last utc time that the pack had a (key, value) pair modified. |

## Pack Meta Table

* Potential keys: description, slack_channel, wiki.

| pack_name                                                   | last_modified_time_utc                                | field_value                        | field_name                                               | region                                            | environment                                                  |
| ----------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------- | -------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| The name of a pack. Foreign key to pack_name in Pack Table. | The last utc time that this row's value was modified. | The value of the `Metadata` field. | The name of the `Metadata` field, such as slack_channel. | The region this (key, value) pair corresponds to. | The environment (prod, preprod, stage, dev) that this (key, value) pair corresponds to. |

## Service Table

| service_name                          | last_modified_time_utc                                       |
| ------------------------------------- | ------------------------------------------------------------ |
| The name of the service. Primary key. | The last utc time that the service had a (key, value) pair modified. |

## Service Meta Table

* Potential keys: description, tier, pack, repo, wiki, slack_channel.

| service_name                                                 | last_modified_time_utc                                | field_value                        | field_name                                      | region                                            | environment                                                  |
| ------------------------------------------------------------ | ----------------------------------------------------- | ---------------------------------- | ----------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| The name of a service. Foreign key to service_name in Service Table. | The last utc time that this row's value was modified. | The value of the `Metadata` field. | The name of the `Metadata` field, such as wiki. | The region this (key, value) pair corresponds to. | The environment (prod, preprod, stage, dev) that this (key, value) pair corresponds to. |