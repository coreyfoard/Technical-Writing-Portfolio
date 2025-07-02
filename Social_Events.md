# Social Events SQS Data Pipeline: `<env>-offer-social-events`

[SQS pipeline](https://sqs.us-east-1.amazonaws.com/292095839402/<env>-offer-social-events)
[Lucidchart](https://lucid.app/lucidchart/4454f08e-f9c9-4583-ad98-856e970f1669/edit?page=0_0&invitationId=inv_51d5fe3a-c0ae-41bb-ac57-c9e2ab08319c#)

## What is this SQS pipeline

We need a way to send social offers' data downstream for debugging and reporting purposes.
`Offer-service` sends messages to the `<env>-offer-social-events` SQS queue with different types
of social events. This queue is then consumed by Kinesis than then sends the data to a Snowflake table.

### Types of social events

1. **GROUP_UNLOCK**: occurs when a user meets the social offer's requirement, this event can only be triggered
by the host of the group (i.e. the person that created the group).
2. **USER_JOIN**: occurs when a user joins an active group, this event cannot be triggered by the host but by all
other participants in the group. A user can join by choice or by an invitation from another user.
3. **USER_PARTICIPATE**: occurs when a participant of the group redeems and fulfill their part, this event cannot
be triggered by the host or by the last participant redeeming.
4. **GROUP_COMPLETE**: occurs with the last participant redeeming in the group, that means everybody else have
completed (redeemed) their part. This event is FIRST triggered by the last participant redeeming, this in turn, will 
trigger the same type of event for the rest of the participants.
5. **USER_DROP**: occurs when a participant leaves the group, this event can only be triggered by the user leaving
or by the host of the group. A user can leave a group by choice or if the host removes them from the group.

### Message payloads

The data sent in each message is formed with the `SocialEventMessage` class.

```kotlin
data class SocialEventMessage (
    // Required
    val eventType: SafeEnum<SocialEventType> = SocialEventType.UNKNOWN.safe(),
    val userId: UserId,
    val groupId: GroupId,
    val offerId: OfferId,
    val eventTime: Long,

    // optional params
    val receiptId: String,
    val groupStartTime: Long,
    val groupExpirationTime: Long,
    val completionPoints: Long,
    val unlockingSource: SafeEnum<UnlockingSource>,
    val joiningSource:  SafeEnum<JoiningSource>,
    val numParticipants: Int,
    val payerId: String,
)
```

### Event time in message payload

It is important to remark that the `eventTime` come from different sources depending on the type of event:

- GROUP_UNLOCK: in the present time, a user can unlock a group only by scanning a receipt that meet the offers'
  requirement, hence `eventTime` is `receipt.scanTime` from the host. Coming soon: social groups will be unlocked
  in other ways.
- USER_JOIN: `eventTime` is generated at the time of the event.
- USER_PARTICIPATE: `eventTime` is `receipt.purchaseTime` from a participant in the group.
- GROUP_COMPLETE: `eventTime` is `receipt.purchaseTime` from the last participant in the group.
- USER_DROP: TBD

### Example of `GROUP_UNLOCK` event type message

```json
{
    "eventType": "GROUP_UNLOCK",
    "userId": "54c94233e4b021f94c49e00b",
    "groupId": "01GG89729BA4ZEGESCR03P00VY",
    "offerId": "01GG88RBNK4TQ4QT9PRGRJZ20H",
    "eventTime": 1669790059000,
    "groupTimeEnd": 1669790059000,
    "receiptId": "46a94233e4b122f94c49e89c"
}
```
