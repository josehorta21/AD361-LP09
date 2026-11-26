# Step 2. General prompt

## Original prompt

```text
Create tests for a vacation booking application.
```

## Complete LLM response

## Vacation booking acceptance tests

These tests cover a traveler searching for lodging, reserving a room, paying, and canceling. They are ready for manual execution or conversion into automated acceptance tests. They have not been run against an application.

**Test setup.** Seed a property with one room that accommodates two guests and costs USD 120 per night. Apply 10% tax and no fees. The room is available from June 10 through June 15, 2027, with checkout dates excluded from occupancy. Use a payment sandbox that can approve, decline, or delay a response. Reset inventory before each test.

| ID | Scenario and action | Expected result |
|---|---|---|
| V01 | Search the property’s destination for two guests, checking in June 10 and out June 13. | The room appears. The dates represent three nights, and the quoted total is USD 396.00. |
| V02 | Repeat the search with checkout equal to check-in, then with checkout one day before check-in. | Each search identifies the invalid checkout date and prevents booking. |
| V03 | Search for three guests, then repeat for two. | The two-person room is excluded for three guests and included for two. |
| V04 | Seed a confirmed stay for June 10–13. Search June 12–14, then June 13–14. | The overlapping stay is unavailable. A new guest can check in on June 13. |
| V05 | Book the June 10–13 stay with an approved payment. Reload the confirmation and open the traveler’s booking history. | Both views show the same booking reference, dates, guests, and USD 396.00 total. Exactly one payment was captured, and the room is unavailable for those nights. |
| V06 | Submit the same booking with a declined payment. Retry with an approved payment. | The decline is shown without a confirmed booking or captured charge. The retry produces one confirmed booking and one successful charge. |
| V07 | Have the payment sandbox approve the charge but delay its response until the browser times out. Retry the same checkout. | The traveler can recover the booking status. The final records contain one confirmed booking and one captured charge. |
| V08 | Have two travelers open checkout for the last room. Submit both bookings at the same time. | Exactly one traveler receives a confirmed booking. The other sees that the room is unavailable and has no captured charge. |
| V09 | Sign in as a second traveler and open the first traveler’s booking detail URL. Attempt to cancel it. | The application denies access, reveals no booking details, and leaves the reservation and payment unchanged. |
| V10 | Book a rate that offers a full refund until 48 hours before check-in. Set check-in to June 10 at 3 p.m. property time and cancel on June 8 at 2:59 p.m. Then refresh and retry the cancellation. | The booking is canceled, the room returns to inventory, and exactly one USD 396.00 refund is issued. |
| V11 | For the same rate, which becomes nonrefundable after the cutoff, start cancellation on June 8 at 3:01 p.m. property time. Review the terms and back out. | The preview shows a USD 0.00 refund. Backing out leaves the booking confirmed and the room reserved. |
| V12 | Complete the search and booking flow using only a keyboard. Leave a required field empty on the first submission, then correct it. | The traveler can select dates, identify the field in error, correct it, submit payment, and reach confirmation without using a pointer. |

## Evaluation against North Vacation Booking requirements

| AI-generated idea | Assessment | Reason and correction |
|---|---|---|
| V01 · Three-night stay | Useful date example, incorrect price | June 10–13 spans three reserved nights. For two guests, the expected total is 3 × $200 + $50 = **$650**. The suggested $120 rate, 10% tax, and zero fees came from its invented setup. Evaluate the reservation total directly. |
| V02 · Equal and reversed dates | Useful | Both inputs violate the date-order rule. Submit each as a separate booking attempt and require an invalid-booking error. A field-specific checkout message and a search workflow are extra assumptions. |
| V03 · Capacity filter | Unsupported | The stated booking range includes both two and three guests. A two-person property limit and a search filter need separate requirements. Exercise the actual guest boundaries with 0, 1, 12, and 13 guests. |
| V04 · Overlapping and adjacent stays | Useful, incomplete expected result | With June 10–13 already reserved, June 12 overlaps and must trigger an **unavailable error**. June 13–14 can be reserved if June 13 is available. For two guests its one-night total is $250. |
| V05 · Payment and booking history | Incorrect and unsupported | The $396 total is incorrect for the two-guest example referenced by the setup. The three-night booking should cost $650. Payment capture, reference numbers, and matching history views require additional specifications. |
| V06 · Declined payment and retry | Unsupported | The requirements give no payment states or retry rules. A declined card cannot stand in for invalid booking information. Use an out-of-range guest count to test the specified error. |
| V07 · Recover after a payment timeout | Unsupported and vague | No payment or recovery contract is supplied. “Recover the booking status” also gives the tester no recovery action, screen, or observable state to check. |
| V08 · Two travelers compete for a room | Useful risk, partly unsupported | Duplicate use of one night is worth exploring. A reproducible requirements test can start with one night already unavailable and verify that the second request produces an unavailable error. Simultaneous checkout and charge handling need separate specifications. |
| V09 · Another traveler’s reservation | Unsupported | User accounts, authorization rules, booking-detail links, and cancellation are outside the stated requirements. The proposed access decision cannot be graded from this specification. |
| V10 · Cancel just before a refund cutoff | Unsupported | The 48-hour policy, local check-in time, refund, and cancellation behavior all lack a requirement. The proposed $396 refund also inherits the example’s invented price. |
| V11 · Back out of cancellation | Unsupported | A nonrefundable rate, refund-preview screen, and “back out” flow all depend on a cancellation policy the assignment never supplies. |
| V12 · Keyboard-only booking and recovery | Vague and partly unsupported | An empty required field could exercise invalid input, but the case never identifies the field. Replace that action with entering 0 guests and checking for an invalid-booking error. Keyboard navigation and payment are separate topics. |

### Missing coverage

The response never tests the 1-night and 31-night limits together with their outside neighbors (0 and 32), or the 1–12 guest limit with 0 and 13 guests. Its pricing assumptions obscure the requirements for the $200 nightly rate, exactly one $50 cleaning fee per valid reservation, and $25 per night for **each** guest above five. For example, on two available nights, five, six, and seven guests should cost $450, $500, and $550 respectively.

V02 and V04 identify date ordering and checkout-day exclusion. The test suite still needs a booking with one unavailable night strictly inside a longer stay, with the surrounding nights available. It should distinguish an **invalid-booking error** for invalid inputs from an **unavailable error** for valid dates that overlap unavailable inventory. The assignment specifies the meaning of each error and leaves the exact wording open.
