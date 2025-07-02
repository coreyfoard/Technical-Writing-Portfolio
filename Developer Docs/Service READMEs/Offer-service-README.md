# Offer Service

This service is responsible for calculating user eligibility for Offers,
as well and determining which Offers should be redeemed on a given receipt.

## Development

1. Install Java 17 (Amazon Corretto recommended)
2. Configure `aws-cli` per the [onboarding guide](https://fetchrewards.atlassian.net/wiki/spaces/DOB/pages/1695056059/AWS+CLI).
3. Perform login via your terminal `aws sso login`.
4. Configure artifactory credentials in your path or in an `local.properties` file in the project root.
1. Path Variables:
	1. Username variable `ARTIFACTORY_USER`
	2. Password variable `ARTIFACTORY_PASSWORD` (retrieve Artifactory API Key [here](https://fetch.jfrog.io/ui/admin/artifactory/user_profile))
2. `local.properties`
	1. Username key `artifactory.username`
	2. Password key `artifactory.password` (retrieve Artifactory API Key [here](https://fetch.jfrog.io/ui/admin/artifactory/user_profile))
5. Configure the environment variable `MICRONAUT_ENVIRONMENTS=local` in your run configuration
6. Latest Gradle (or just use included wrapper!)
7. **ALWAYS Build, run tests, and preferably build production artifact prior to PR submission** (`./gradlew clean shadowJar`)
Note: we'll know :wink:
8. If linting fails, run `./gradlew spotlessApply`

Not sure how to install/manage the JDK?
Refer to the following:

```shell
#
# E.g.

# Install asdf-vm for jdk/toolchain version management
# ref: https://asdf-vm.com/guide/getting-started.html#_1-install-dependencies
# Note there are required steps outlined in the above link,
# brew install is not sufficient.
brew install asdf

# Add java plugin
asdf plugin add java

# Install JDK + activate
asdf install java corretto-17.0.5.8.1
cd offer-service
asdf local java corretto-17.0.5.8.1
```

Note that you need to ensure your jdk is correct before running any gradle commands. (If you run gradle with the wrong
JDK, manually clear your local gradle cache.)

---

## Deployment

Refer to [Deployment Guide](./docs/Deployment_Steps.md)

---

## Architecture

`offer-service` uses user demographic and account information, purchase history,
and incoming receipt data to determine eligibility and to evaluate redemption.

### Service Dependencies

1. `purchase-history-service`
2. `identity-service`
3. `holdouts-service`
4. `golatac-catalog`

#### purchase-history-service

Query for a user's past year of purchase history in version 2 format, encoded
as msgpack.

#### identity-service

Query for a user's account information, including these relevant data:

1. State
2. Zip
3. Age
4. Gender
5. Account Creation Date
6. Segments
7. Loyalty Programs
8. Holdout "Optouts" (whitelist)

#### holdouts-service

Query with all possibly relevant offers to see which the user should be heldout
from, making sure to account for loyalty programs and whitelisting.

#### golatac-catalog

Query for the Fidos associated with any barcodes saved a part of an offer's
eligibility and action requirements.

---

## Eligibility Flow

Eligibility for each active offer is calculated for a user each time it is requested.
The following is a high-level view of the steps that take place when determining the
latest eligibility:

1. Request necessary information concurrently
	1. User account information from `identity-service`
	2. User offer usages from `offer-service` Postgres
	3. User purchase history from `purchase-history-service`
	4. Most recent eligibility snapshot from `offer-service` Redis
	5. Active offers from `offer-service` Redis
2. Calculate eligibility status for every active offer
3. Update eligibility snapshot
	1. Remove invalid offers (long past relevant dates)
	2. Remove offers with failed a disqualifying criteria
	3. Add newly eligible offers
	4. Save latest snapshot to Redis
4. Sort and filter offers based on stackability
	1. Sort offers by start day, then by value
	2. Add all stackable offers to return set
	3. For each non-stackable offer, add to return set if it does not overlap another offer
5. Filter offers for user visibility
	1. Discard "dark" offers that have made no progress
	2. ... More to come?
6. Return list of eligibility offers, with user-specific start/end times

---

## Redemption Flow

Redemption necessarily recalculates eligibility, then applies the redemption criteria for
all eligible offers to the provided receipt.The following is a high-level view of the steps
that take place when determining the offers to redeem for a user on any given receipt:

1. [Eligibility Flow](#eligibility-flow) steps 1-4
2. Apply redemption criteria to current receipt and purchase history (in order)
3. If `finalize = true` on request:
	1. Upsert offer usages for user
	2. Save detailed contribution information to DynamoDB
	3. Update "recent progress" cache for any relevant multi-transaction offers
4. Calculate benefits for each successful redemption
5. Return number of points that should be awarded, bucketed by line item and offer id

---

## Eligibility Criteria

Eligibility is separated into a number of separate steps that allow for greater customization
of offers, as well as to provide more information as to cause of an eligible or ineligible result.

Each criterion can broadly be categorized into two distinct categories based on how they influence
any offer from a user's perspective: **standard** and **disqualifying**. If details are not listed
below, they can be found on the
[wiki](https://fetchrewards.atlassian.net/wiki/spaces/OP/pages/1651441762/Offer+Business+Overview).

### Standard Criteria

Standard criteria only influences whether a user initially becomes eligible and receives a user specific
offer instance. Failing to meet a standard criteria after a user has already become eligible has no
effect on the user's ability to see and redeem the offer.

- Purchase History
- Gender
- Postal Code
- Time (eligibility window)
- Duration
- New User
- Lapsed User

[TODO]: # (Details for each standard criterion)

### Disqualifying Criteria

Disqualifying criteria influences whether a user is eligible regardless of the user's current eligible state.
Failing to meet a disqualifying criteria after a user has already become eligible will remove the
offer from the user's view and prevent them from redeeming until they have successfully met the criteria again.

- Age
- Expiration
- State
- Segments
- Max Uses
- Loyalty
- Required Redemption
- Excluded Redemption

[TODO]: # (Details for each disqualifying criterion)

---

## Redemption Criteria

In progress, see [wiki](https://fetchrewards.atlassian.net/wiki/spaces/OP/pages/1651441762/Offer+Business+Overview).

[TODO]: # (Details for each redemption criterion)

---

## Benefits

In progress, see [wiki](https://fetchrewards.atlassian.net/wiki/spaces/OP/pages/1651441762/Offer+Business+Overview).

[TODO]: # (Details for each benefit)

---

## Coding Practices

### Safe Enumeration Encoding

To avoid loss of information during an active deploy, in the event a deployment revert, and to ensure enumeration
omission is properly handled, all enumeration properties on data types should be wrapped in the SafeEnum class.
The signature of this class requires that you check for the existence of a valid value prior to using it, and
successfully decodes all strings without losing any information.

[TODO]: # (Details about identity caching)

[TODO]: # (Details about SOA and iterators)

[TODO]: # (Details about FetchTimeObject)

---
