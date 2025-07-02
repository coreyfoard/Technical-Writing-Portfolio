# Social Offers ("Tango Offers")

Unlock and complete offers with your friends to get points for every member of your group.
> The main difference between a regular offer and a social offer is that social offers are completed with
> 2+ people redeeming to get awarded points.

## Behavioral Flow

1. Mark an offer as "social"
2. Host unlocks a social group
3. Friend joins as a participant
4. Participant contributes for group progress
5. Groups succeed or fail together

### Mark an offer as "social"

Any non-Lapsed offer (any offer with `offerBehaviorType = GENERAL`) can be marked as social by defining the relevant,
optional configuration property:

```
{
    "social": {
        "duration": 7,              // Number of _days_ group is active
        "minParticipants": 3,       // (Unimplemented behavior) Minimum participants for group completion
        "maxParticipants": 3,       // Maximum participants
        "timeEnd": 1671264529999,   // same as offer.endTime, used to determine unlocking capacity
        "benefit": {
            "type": "FLAT_POINTS",    // Award type. Probably want `FLAT_POINTS`
            "points": 200             // Number of points for each particiant on group completion
            }
        }
    "usesPerOffer": 1, // Determines how many times a user can _unlock_ a social-offer
    "otherOfferProperties": ""
}
```

### Host unlocks a social group

Whenever an eligible user meets the action requirements of the offer, they will simultaneously unlock (creates a group),
fulfill their portion of the offer and becomes the host of the social group. Since their obligation to the group has
already been met, this should incentivize friends joining the offer before purchasing relevant items, since they have a
better chance of completing the group and earning the points.

> NOTE: at this time the only way to unlock a group is by scanning a receipt meeting the offer's requirements, however
> unlocking will be possible in different ways with future implementations.

1. Host meets offer's action requirements on `receiptA`
2. New group starts on day of `receiptA.scanTime` 
3. Group ends on `receiptA.scanTime + social.duration` OR when every participant in the group has redeemed.

### Friend joins as participant

Once a user has started a group by unlocking, any of their friends can opt to join the group from an entry in their
activity feed. They will be also able to join by invitation from the host. Semantically, joining a group issues the
participant a new instance of that offer, with a start time the day that they joined and an end time equivalent to the
group's end time.

### Participant contributes for group progress

After a user has joined a group, they must now make a new valid purchase before group ends, and scan their receipt
before it becomes invalid (leniency time - 14 days post end). Any receipt with a `purchaseDate` before join date
or after the group's end date will not satisfy the criteria and therefore will not contribute for group progress.

### Groups succeed or fail together

As soon as the final participant has contributed to the group, everybody in the group will be awarded with points on
their respective receipts.

If scan leniency have passed, 14 days since the group had ended, no subsequent scans can possibly be valid and the 
group has semantically failed to redeem their social offer. No points are awarded to any participants.

## Derived Definitions and Constraints

### User constraints

- User may not participate in more than one group for a given offer at a time
- User may participate in as many groups as offers' duration allows

### Offer constraints

- Offer must be defined as single-transaction redemption
- Offer `benefit.points` should be 0, if points should only be awarded on group completion
- Offer `usesPerUser` is the number of times a user can participate in a group
- Offer `eligibilityRequirement.duration` is ignored in the backend
- Offer duration (days between startTime and endTime) must be <= 90 days
- Offer `social.duration` _must be_ immutable after the offer has gone live and be <= 30 days
- Offer `social.maxParticipants` _must be_ immutable after the offer has gone live
- Offer `social.timeEnd` _must be_ same as `offer.endTime`
- Offer `social.benefit.points` is the number of points each participant on completion will be awarded

### Derived properties

```
group.timeStart = scan time of host's receipt moved to the start of the day
group.timeEnd = group.timeStart + offer.social.duration (immutable)
group.timeComplete = time when group is completed
participant.timeJoin = time when join occurs
```
