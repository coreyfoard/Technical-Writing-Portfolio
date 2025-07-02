# Reprocessing of Receipts with Redemption Discrepancies

[Jira ticket](https://fetchrewards.atlassian.net/browse/OFF-2030)

## What's the problem

When an offer is updated after it begins (examples below) we see an influx in receipts incorrectly awarding for offers
they have not earned.

## What are we doing?

When a receipt is being redeemed see [Redemption_Flow](https://fetchrewards.atlassian.net/wiki/spaces/OP/pages/3266052348/Redemption+Flow+-+Offer+Service.md).
we loop through each offer and check to see if the user has earned a new offer. When evaluating if the offer is earned,
Offer Service lumps the users Purchase History and the newly submitted receipt data into one bucket and decides if the
bucket fits the requirements of the offer again, for heavier details see [Redemption_Flow](https://fetchrewards.atlassian.net/wiki/spaces/OP/pages/3266052348/Redemption+Flow+-+Offer+Service.md).
If a user scans a receipt that should've earned an offer, but was missed (examples below), since the qualifying receipts data
is in the users Purchase History, the next receipt scanned will incorrectly match to having earned the offer. To correct
the incorrect contributions, we will now reprocess receipts that missed offers (at which point they will correctly award the offer).

## How are we doing it?

* Currently, when looking at the Purchase History + Current Receipt data bucket during redemption, we work through what
actual data has earned the offer.
* Now, when we see an offer is complete (or is of type `ReceiptAndHistoryRedemption`), we gather the data that earned it
and create a list of the receipts it came from.
* Before awarding the offer, we check "Is the current receiptId in the list of receipts that contributed to completing the offer?".
  * __If__ it is found, we continue as normal (award the points for the offer).
  * __Else__, we know the current receipt did not earn the offer (i.e. the attributions would be incorrect). We add the receiptId
  to a set of receiptIds that need reprocessed.
* After checking all complete offers, we continue awarding the earned ones, and in a separate thread via
[GlobalScope.Launch](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/)
reprocess the receipts that missed the offers so that they might (they should) pick up the correct offers.

## Examples

* Offer is created with the following criteria:
  * Multi-transactional
  * Items Required = Cheetos Products
  * Quantity = 2 (user must buy any 2 Cheetos items)
  * Start date = 7/15/2022
  * End date = 7/15/2023
* On 7/16 user buys 2 bags of Cheetos and scans their receipt. However, the barcodes from those Cheetos products were missing
on the offer, so the receipt did not earn the offer.
* On 7/17 the additional barcodes are added to the offer.
* On 7/18 the user buys some unrelated items, and scans their receipt.
  * Before implementation of reprocessing, the offer that should've been awarded on 7/16 would incorrectly be awarded on this receipt.
  From a data perspective, attributions would be messed up.
  * After implementation of reprocessing, we catch that we missed the offer on 7/16 and reprocess the receipt that actually earned it.
* The second receipt completes processing, while the first receipt is reprocessed and correctly awards for the offer.
* Upon fresh opening the app, the user will see their first receipt popup with the offer applied (what a pleasant surprise!).

## How are we measuring success?

We should see a significant drop in missed attributions.
One way to see this is on the [Offer Usages Monitor Grafana](https://fetchrewards.grafana.net/d/y4gNWPZVk/offer-usages-monitor?orgId=1).

## What could go wrong?

In the base-case for receipt-processing, a receipt is processed and then the receipt processor calls offer-service to identify
awards for the receipt. In the case that offer-service calls receipt processor*, we could be introducing an infinite loop.
[Catch-22](https://en.wikipedia.org/wiki/Catch-22_(logic)) (i.e. we see an infinite loop).

* We saw this occur during [inc-35691-unable-to-view-receipts-in-supportal](https://fetchrewards.slack.com/archives/C042BV2DJ6Q/p1663113719059089).
Essentially, somehow (need better insight into how) an eligible receipt snuck through without being awarded. Then, when another
eligible receipt was scanned directly after (most often a duplicate) we enter a case in which two receipts have earned the offer,
but neither have awarded it. This introduced a loop in which either receipt identified the other as the earner, and asked it to
reprocess the other.
* To prevent receipts from identifying each other as the earner and entering a loop, we are modifying the considered purchase
history window from `startDate = Offer Start Date`,`endDate = Offer End Date` to `startDate = Offer Start Date`,
`endDate = Receipt Scanned Date`. This means that when a receipt is processed (whether for the first time or for reprocessing) it
will only look at receipts before it as purchase history. We can be sure this will _not_ affect base case redemption, since there
is no use case in which a user will have just scanned a receipt, and we will want to consider receipts in the future to decide if
an offer is earned.

Example on how this fixes the loop below:

__Before__ moving endDate:

* Receipt A scanned on 7/15 at 11:55AM and earned the offer, but it was missed.
* Receipt B scanned on 7/16 at 12:05AM and earned the offer, BUT we evaluate the users purchase history and see the offer was earned
because of purchase history (without the current receipt's data). So, we ask to reprocess the receipt that earned it in history.
* Receipt A is asked to reprocess, and grabs user's purchase history. Because purchase history is considered up until the offer end date
(in the future) we grab Receipt B and consider it Purchase History. Once again, we see that the offer is earned because of purchase
history (without the current receipt's data) and we ask to reprocess the receipt that earned it.
* Receipt B is asked to reprocess, and we infinitely loop between each receipt identifying the other as earning and asking to reprocess.

__After__ moving endDate:

* Receipt A scanned on 7/15 at 11:55AM and earned the offer, but it was missed.
* Receipt B scanned on 7/16 at 12:05AM and earned the offer, BUT we evaluate the users purchase history and see the offer was earned
because of purchase history (without the current receipt's data). So, we ask to reprocess the receipt that earned it in history.
* Receipt A is asked to reprocess, and grabs user's purchase history. Because purchase history is _only_ considered to be receipts
scanned prior to the current receipt, Receipt B is __not__ considered Purchase History. We see that the offer is earned because of the
current receipt, and award the offer (on the receipt that should've earned the offer).

EVEN THOUGH we have fixed the identified loop case, we have still created alarms to alert if one receipt is asked to reprocess multiple
times by another same one receipt.

### Something breaks in receipt-processing

* We saw this occur during [#inc-35539-high-errors-on-frs-and-other-services](https://fetchrewards.slack.com/archives/C0405PTS9KK/p1661790989742619).
Right after releasing this work, we saw errors in FRS on receipt processing. Initially, we concluded it was because of our deployment
and decided to roll-back offer-service. After some confusing investigating, we identified that it was not our release that caused
the fall of FRS, but a #pack-thames product mapping batch job. Regardless, the investigation gave insights to a couple of improvements
that could be made to catch it if reprocessing failed.
* IF reprocessing fails, offer-service waits in a thread for a response from the reprocesses, and we have alarms on multiple failures
so that we will know.
* Also, given the nature of this work, it is self-correcting in that on the next receipt scan, we will repeat the process and have
the opportunity to fix missed awards again.

## What to do if something does go wrong?

In [FlagConfig.kt](https://bitbucket.org/fetchrewards/offer-service/src/master/src/main/java/com/fetchrewards/config/FlagConfig.kt)
we can set `reprocessInNewArch` to false to turn off reprocessing in the new arch and/or `reprocessInOldArch` for the old (then redeploy).
