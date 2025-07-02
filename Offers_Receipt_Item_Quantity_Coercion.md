# Receipt Item Quantity Coercion

[`Document in Offer-service repository`](https://bitbucket.org/fetchrewards/offer-service/src/master/docs/Offers_Receipt_Item_Quantity_Coercion.md)

## What's the problem

Here's a good read on the inspiration: [item quantities and their legacy representation](https://docs.google.com/document/d/1p6EIdWiCcTEQt_6oe5TSnBlHug6Wbpm0I0tmy3L8RvU/edit#heading=h.p2qa5068fhrw)

In short, item quantities can be represented in a way that is problematic for offer redemption and uniform consumption
in places like purchase histories. We became aware of this problem and need a short term way to address it, as it accounts for
a significant leak in revenue.

## What are we doing?

* In consultation amongst ourselves and other packs, we decided to put a fix in place in the offer service. While we would prefer that this
change be made upstream of the offer-service, that will likely have far-reaching consequences, so we're here for now.
* In order to properly consider those items, where for one reason or another the quantity is less than 1, we are simply coercing
the quantity to 1.
  * We need to do this because we need to be able to properly consider those items in the offer redemption process,
  which is contingent upon a sensible quantity value.

## How are we doing it?

* We've made a `QuantityCoercionConfig` to manage those items whose quantities we will look to set to at least 1. In order to
limit the potential impact of such a change, we are gating this coercion logic with barcode and fido restrictions.
  * Specifically, quantity coercion for invalid quantities will only take place on those items with a barcode or fido that falls
  into the set enumerated in the QuantityCoercionConfig.
  * The set of barcodes and fidos that are allowed to be coerced is configurable within the `application.yml`, found [here](https://bitbucket.org/fetchrewards/offer-service/src/master/src/main/resources/application.yml)
    or with the environment variables `FLAG_QUANTITY_BARCODES_FOR_QUANTITY_COERCION` and `FLAG_QUANTITY_FIDOS_FOR_QUANTITY_COERCION` e.g. `FLAG_QUANTITY_FIDOS_FOR_QUANTITY_COERCION=fido_1,fido_2`
* Receipt item coercion happens during redemption requests, actuated in the controller layer.
