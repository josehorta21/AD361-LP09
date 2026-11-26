# Learning Practice 9 · Enhancing Testing with Large Language Models

North Vacation Booking | AD361 – Application Testing

## 1. Initial test ideas

My starting points were prices, availability, ratings, and comments. The table turns my pricing and availability ideas into concrete checks using the assignment rules. I would ask whether ratings and comments are in scope before writing an expected result for either feature.

| ID | Requirement | Input or action | Expected result |
| --- | --- | --- | --- |
| H1 | Base price and one cleaning fee | Request June 10–13, 2027, for two guests with all three nights open. | Accept at 3 × $200 + $50 = $650. |
| H2 | Extra guests are charged per night | Request June 10–13 for seven guests with all nights open. | Accept at 3 × ($200 + 2 × $25) + $50 = $800. |
| H3 | Each valid reservation has its own cleaning fee | For two guests, make two separate, available one-night reservations on June 10–11 and June 15–16. | Each reservation costs $250. Together they cost $500, including two cleaning fees. |
| H4 | All requested nights must be available | Request June 10–13 for two guests with all three requested nights blocked. | Reject the request with an unavailable error. |
| H5 | One available night cannot make up for an unavailable night | Request June 10–12 for two guests. Only the night of June 10 is open. | Reject the entire two-night request with an unavailable error. |

## 2–4. Prompts, complete responses, and review decisions

- [General prompt, complete response, and review of V01–V12](general-prompt.md)
- [Contextual prompt, complete response, and review of T1–T8](contextual-prompt.md)

## 5. Comparing the prompts

The contextual prompt produced more useful tests. The general prompt's V01 looks precise because it gives a total of $396 for three nights and two guests. But the model picked a $120 rate, added tax, and left out the cleaning fee. For this assignment the expected total is 3 × $200 + $50 = **$650**. Using V01 as written would flag a correct booking as a defect.

Giving the model the actual surcharge rule also changed what it tested. T3 holds the dates constant and changes the guest count from five to six. The two-night total goes from $450 to $500, which makes it easy to spot a system that charges the extra $25 only once. One weakness remains in the contextual response: T4 checks equal dates, but never checks checkout *before* check-in. A program that rejects equal dates could still accept reversed ones. The general prompt actually caught both in V02.

**Comparison with the initial ideas.** My H4 and H5 place the blocked night inside the stay. T1 added a case I hadn't listed: block only the night that starts on checkout day and verify that the booking still succeeds. Going the other way, H3 checks two separate valid reservations and two cleaning fees. Neither model response checked that each separate reservation gets its own $50 fee.

## 6. Final test cases

Each row is a separate attempt on the same test property. Use the dates shown in 2027. Reset the property's availability before every attempt so one reservation cannot affect the next test. “Open” means every night in the requested stay is available unless the row states otherwise. Submit the reservation with the listed dates and guest count, then check the result.

| ID | Requirement | Preconditions | Check-in → checkout; guests | Expected result | Source |
| --- | --- | --- | --- | --- | --- |
| TC-01 | Minimum stay and guest count; checkout excluded | June 10 open; night of June 11 unavailable | June 10 → June 11; 1 | Accept one night for $250. Only the night of June 10 is needed. | AI-generated and verified |
| TC-02 | Maximum stay and guest count; surcharge on each extra guest each night | Open | June 1 → July 2; 12 | Accept 31 nights. Seven extra guests add $175 per night. Total 31 × $375 + $50 = $11,675. | AI-generated and verified |
| TC-03 | No surcharge at five guests; one cleaning fee per stay | Open | July 7 → July 9; 5 | Accept at 2 × $200 + $50 = $450. | AI-generated and revised |
| TC-04 | Sixth guest incurs a nightly surcharge | Same dates open on a fresh run | July 7 → July 9; 6 | Accept at 2 × ($200 + $25) + $50 = $500. This is $50 more than TC-03. | AI-generated and revised |
| TC-05 | Stay must be at least one night | No conflicting reservations | June 15 → June 15; 5 | Reject with an invalid booking information error. The stay has zero nights. | AI-generated and revised |
| TC-06 | Checkout must follow check-in | No conflicting reservations | June 15 → June 14; 5 | Reject with an invalid booking information error. | AI-generated and revised |
| TC-07 | Stay must be at most 31 nights | Open | June 1 → July 3; 5 | Reject the 32-night request with an invalid booking information error. | AI-generated and verified |
| TC-08 | Guest count must be at least one | Open | July 12 → July 13; 0 | Reject with an invalid booking information error. | AI-generated and verified |
| TC-09 | Guest count must be at most 12 | Open | July 12 → July 13; 13 | Reject with an invalid booking information error. | AI-generated and verified |
| TC-10 | Every requested night must be available | June 20 and 22 open; night of June 21 unavailable | June 20 → June 23; 5 | Reject the entire stay with an unavailable error. | AI-generated and verified |
| TC-11 | Cleaning fee applies separately to each valid reservation | Same property; June 10 and June 15 both open | Submit June 10 → June 11 for 2 guests, then June 15 → June 16 for 2 guests | Both accepted. Each one-night reservation totals $250. The two totals add to $500. | Human-generated |
| TC-12 | Two extra guests for three nights | Open | June 10 → June 13; 7 | Accept at $800: $600 for three nights, $150 for six extra guest-nights, and one $50 cleaning fee. | Human-generated |
| TC-13 | No requested night is available | Nights of June 10, 11 and 12 blocked | June 10 → June 13; 2 | Reject with an unavailable error. | Human-generated |

TC-11 through TC-13 develop the tester's original pricing and availability ideas from H3, H2 and H4. The concrete dates and calculations were worked out with AI assistance.

TC-03 and TC-04 split T3 into separately repeatable cases. TC-06 takes the reversed-date idea from V02 and supplies specific dates, a valid guest count, and the required error category.

**Execution status.** These are test designs. The [starter booking form](airbnb-clone-ts/src/pages/AddBooking.tsx) accepts a rental, a single date and a guest count. Its number input has a minimum of 1 and no maximum. A tester can check whether the browser stops zero guests or allows 13 to be submitted, then inspect the [bookings list](airbnb-clone-ts/src/pages/Bookings.tsx). The full cases above need a build that accepts both dates. Pricing checks also need an observable total, and availability checks need controllable nightly inventory. The [booking context](airbnb-clone-ts/src/context/BookingContext.tsx) only appends the submitted data to a list. No application test result is claimed here.

## 7. Reflection

1. **What did the LLM speed up?** It turned the limits into a useful first set of inputs, especially the 31-night stay and the 32-night request that crosses into July.
2. **What did it miss or invent?** The general response invented taxes, payment processing, and a refund cutoff. The contextual response missed the reversed-date case even though the prompt gave the date-order rule.
3. **What knowledge was needed to evaluate it?** The checkout date ends the stay, so June 1 to July 2 is 31 nights. To check the largest total, the surcharge applies to seven of the twelve guests every night, while the cleaning fee applies once to the whole stay.
4. **What could go wrong without review?** If a tester used $396 as the expected price in V01, the developer could be asked to “fix” the correct $650 total. The same suite might still miss a real defect, like accepting 13 guests.
5. **What decisions belong to the tester?** The tester chooses the expected result from the requirements, verifies the arithmetic, and checks that the setup can isolate the rule being tested. For example, a blocked night inside the stay and a blocked night starting on checkout day should produce different outcomes.

[Original assignment instructions](Readmd.md) · [Starter application](airbnb-clone-ts/)
