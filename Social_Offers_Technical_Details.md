# Technical Implementation Details

## In this article

**[Database](#database)** 

**[Social events messages](#social-events-messages)**

**[Public API](#public-api)**

---

## Database

### Data Schema

There are currently  two Postgres tables containing social-offers data. The first contains the group's information, while the second has each participant's information.

```postgresql
CREATE TABLE social_groups
(
    group_id         varchar     NOT NULL,
    host_id          varchar     NOT NULL,
    offer_id         varchar     NOT NULL,

    time_start       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    time_end         TIMESTAMPTZ NOT NULL,

    time_complete    TIMESTAMPTZ NULL,
    num_participants int         NOT NULL,

    PRIMARY KEY (group_id)
);
```
_Primary key_
`CREATE INDEX CONCURRENTLY social_groups_by_offer ON social_groups (offer_id);`

```postgresql
CREATE TABLE social_participants
(
    group_id       varchar     NOT NULL,
    participant_id varchar     NOT NULL,
    receipt_id     varchar     NULL,

    time_join      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    time_redeem    TIMESTAMPTZ NULL,
    PRIMARY KEY (group_id, participant_id)
);
```
_Primary key_
`CREATE INDEX CONCURRENTLY social_participants_by_id ON social_participants (participant_id);`

#### SQL Semantics

Use the following SQL semantics when accessing the database.

```
group_id = ULID
hostId = user that unlocked the group
group.time_start = host's receipt.scanTime starting at the end of the day central time
group.time_end = group.time_start + offer.social.duration (immutable)
group.time_complete = receipt.purchaseTime from last participant redeeming
particpant.time_join = timestamp when user joined
participant.time_redeem = receipt.purchasetime
```

### Eligibility and Snapshot

Eligibility works the same as regular offers, however social-offers are not visible to a user unless they unlock a social-group. 

To check if a user's eligibility to receive an offer that is currently locked, use the `GET snapshot` API with the query param `support=true`.

### Group unlocking and joining

#### Unlocking

A user can unlock a group if they:
- Are eligible for the offer.
- Meet the offer's redemption requirements.
- Are not participating/hosting in another social group with the same offer.

#### Joining

A user can join a group if:
- User is not participating/hosting another social group with the same offer.
- Group has not expired or is complete.

NOTE: A user may participate/host in as many groups as the offers' duration allows.

### Participant contribution and group completion

Participants contribute to the group by meeting the offer's requirement. Only after every participant contributes do participants receive points. 

All other participants will have their redeeming
receipts triggered into reprocessing, and will receive points for those relevant receipts.

TODO: social: explain reprocessing minutiae. specifically, pure and impure SocialRedemption.Completion TODO: social:
workshop 'pure' terminology

[back to top](#technical-implementation-details)

---

## Social events messages

### Data Pipeline

Offer-service sends messages to SQS `<env>-offer-social-events` to inform data team of social events.

**Group unlock**: user scans and becomes host of a social group.
```
{
  "eventType": "GROUP_UNLOCK",
  "userId": host id, 
  "groupId": from unlocked group,
  "offerId": social offer id,
  "eventTime": receipt.scanTime in miliseconds,
  "groupExpirationTime": group.timeStart + group.duration in miliseconds,
  "numParticipants": the min between minParticipants and maxParticipants in social config,
  "unlockingSource": "REDEEM_RECEIPT"
}
```

**User join**: user joins an active social group.
```
{
  "eventType": "USER_JOIN",
  "userId": id of user joining, 
  "groupId": group id that user joined,
  "offerId": social offer id,
  "eventTime": time when joining,
}
```

**Participant contribution**: participant of a group contributes their part. 
```
{
  "eventType": "PARTICIPANT_CONTRIBUTION",
  "userId": id of user that is contributing to the group, 
  "groupId": group id where user is contributing to,
  "offerId": social offer id,
  "eventTime": receipt.purchaseTime in miliseconds,
  "receiptId": receipt id
}
```

**Group complete**: last participant redeeming and completing group for everybody
```
{
  "eventType": "GROUP_COMPLETE",
  "userId": id of last user contributing/redeeming, 
  "groupId": group id where user is contributing to,
  "offerId": social offer id,
  "eventTime": receipt.purchaseTime in miliseconds,
  "receiptId": receipt id,
  "completionPoints" = points under `offer.social.benefit.points`
  "payerId": payerId from the offer object.
}
```

### Websocket messages

Offer-service sends messages to WES service (Websocket Event Service) so that backend can communicate when
a social-event has occurred. This lets mobile update all participants' group information.

These social-events contain the following information:
```
{
  "eventType": "SOCIAL_OFFER",
  "eventDate": "1670824800000",
  "userId": "5e0e423caec96d0f0de8c7e4",
  "attributes": {
    "groupId": "01GM70DECPZM9XECZTCZZ0WD1D",
    "offerId": "01GKQY4KEAESBDCZ5DQZXEA3J3",
    "reason": "GROUP_UNLOCK" // this is different for each event.
  }
}
```

Different values for `attributes.reason` for each type of event:

|VALUE|WEBSOCKET EVENT EXPECTED
|--|--|
|GROUP_UNLOCK|Sent when a user unlocks a social group.|
|USER_JOIN|Sent when a new user joins an active social group.|
|PARTICIPANT_CONTRIBUTION|Sent when a participant of a social group is fulfilling their part.|
|GROUP_COMPLETE|Sent when last participant redeems completing the social group.|

[back to top](#technical-implementation-details)

---

## Public API

### Join a group

Endpoint response with an empty body and 200 code if join was successful. 
If user can't join, it will return a 400 with a reason of failure.
```
POST {offer-service}/user/{userId}/social/{groupId}/join
```

### Get user's groups

Endpoint returns only all ACTIVE and LATENT groups for the given user.
```
GET {offer-service}/user/{userId}/social
```

### Get group for user with joinability

Endpoint returns a group with information if given user could potentially join the given group.
```
GET {offer-service}/user/{userId}/social/{groupId}
```

[back to top](#technical-implementation-details)

---