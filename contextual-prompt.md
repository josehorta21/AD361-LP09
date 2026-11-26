# Contextual prompt and response

## Prompt as submitted

```text
Act as a software tester helping me design black-box tests for North Vacation Booking. I will verify the calculations, decide which cases to keep, and check whether the app can run them. Give me exactly eight candidate tests in a Markdown table with ID, requirement, preconditions, concrete input or action, and expected result. Use boundary value analysis for length of stay and guest count. Consider the pricing change from five to six guests, checkout-day counting, and a single unavailable night within a stay. Where a booking is valid, show the calculation and total. Use dates in June and July 2027. Treat each row as independent.

Requirements:
- A reservation lasts 1 to 31 nights and includes 1 to 12 guests, inclusive.
- Each night costs $200. Each valid reservation has a one-time $50 cleaning fee. Each guest over five adds $25 per night.
- Check-in must be before checkout. The checkout date is not a reserved night.
- Every requested night must be available. Invalid booking information produces an error. Unavailable dates produce an unavailable error.

Use only these requirements. Do not invent features, exact error messages, or business rules. If a detail needed to decide an expected result is missing, identify it.
```

## Complete response as returned

| ID | Requirement | Preconditions | Concrete input or action | Expected result |
|---|---|---|---|---|
| T1 | A stay of 1 night with 1 guest is valid. Checkout day is excluded. | The night of June 10 is available. The night of June 11 is unavailable. | Book June 10–11, 2027, for 1 guest. | Accepted. Only June 10 is reserved. Total = 1 × $200 + $50 = **$250**. |
| T2 | A stay of 31 nights with 12 guests is valid. | All requested nights are available. | Book June 1–July 2, 2027, for 12 guests. | Accepted for 31 nights. Seven guests incur the nightly surcharge. Total = 31 × ($200 + 7 × $25) + $50 = **$11,675**. |
| T3 | The nightly guest surcharge starts with the sixth guest. | The nights of July 7 and 8 are available at the start of each run. | Book July 7–9, 2027, twice in separate runs, first for 5 guests and then for 6. | Both accepted. For 5 guests, 2 × $200 + $50 = **$450**. For 6 guests, 2 × ($200 + $25) + $50 = **$500**. |
| T4 | Check-in must precede checkout. | Use 5 guests. | Enter June 15, 2027, for both check-in and checkout. | Invalid booking information error. The stay is 0 nights. |
| T5 | A stay longer than 31 nights is invalid. | All requested nights are available. | Request June 1–July 3, 2027, for 5 guests. | Invalid booking information error. The stay is 32 nights. |
| T6 | At least 1 guest is required. | The night of July 12 is available. | Request July 12–13, 2027, for 0 guests. | Invalid booking information error. |
| T7 | At most 12 guests are allowed. | The night of July 12 is available. | Request July 12–13, 2027, for 13 guests. | Invalid booking information error. |
| T8 | Every requested night must be available. | The nights of June 20 and 22 are available. The night of June 21 is unavailable. | Request June 20–23, 2027, for 5 guests. | Unavailable error. June 21 is a requested night, so the stay cannot be booked. |

## Review decisions

These decisions evaluate the test designs against the supplied requirements. The expected results are predictions. No booking was executed for this review.

| Idea | Decision | Reason | Correction or verification |
|---|---|---|---|
| T1 | Accept | An unavailable checkout night makes an off-by-one error observable. One night and one guest satisfy both lower limits. | Keep both availability settings. The one reserved night costs $200 and the one-time fee brings the total to $250. |
| T2 | Accept | From June 1 up to July 2 there are 30 June nights plus July 1. At the maximum party size, seven guests pay the surcharge each night. | Verified $375 per night × 31 = $11,625, plus $50 = $11,675. If this combined case fails, vary the two limits independently to diagnose it. |
| T3 | Revise | Both totals are correct. Two distinct bookings in one row need separate observations so a failure identifies the affected guest count. | Record runs T3a and T3b separately. Start each with July 7 and 8 available. Record $450 for five guests and $500 for six. Their $50 difference equals two nights of the sixth guest's surcharge. |
| T4 | Revise | Equal dates test zero nights. The original action only enters dates, which leaves the point at which to observe validation unclear. | With five guests, attempt to confirm June 15 to June 15 and verify an invalid-booking error. Test checkout before check-in separately. |
| T5 | Accept | The request is precisely one night above the maximum and availability and guest count are valid. | Verified June 1 through June 30 plus July 1 and 2 gives 32 requested nights. Expect an invalid-booking error. |
| T6 | Accept | Zero guests is immediately below one. The available one-night stay isolates the guest input. | Keep the expected invalid-booking error. The requirements specify no exact wording. |
| T7 | Accept | Thirteen guests is immediately above twelve. The date and availability setup isolates the upper guest limit. | Keep the expected invalid-booking error, even if the starter allows the value. |
| T8 | Accept | An unavailable middle night tests whether the app checks the whole stay. Both surrounding nights are available. | Keep the setup and require the unavailable error. June 23 is excluded from the request. |

## Coverage gaps

The response covers all listed rules, but the date-order test only exercises equality. Add a reversed-date case with check-in July 16, 2027, checkout July 15, 2027, and five guests. Confirmation should produce an invalid-booking error. The candidates cover values immediately outside the limits and the limits themselves. Additional just-inside checks could exercise 30 nights, two guests, and eleven guests. The requirements do not establish which error takes precedence if the same request has both invalid input and unavailable nights. Each rejection case above isolates one reason.

## What the starter UI can exercise

The routed booking page at `/bookings/add` exposes one date field and a guest count. The bookings list displays one date and a guest count. There is no checkout field, booking total, or per-night availability control or display. Consequently, the full date-range, pricing, and availability checks in T1–T5 and T8 cannot be performed through this starter UI. Availability in these designs is required test data. The prompt did not specify how to prepare it.

The guest field sets a minimum of one and has no maximum. T6 can be partially exercised by entering July 12 and zero guests and attempting confirmation. Native form validation should block submission. For T7 the same UI can be probed with 13 guests, but the form defines no upper-bound check. These single-date probes cannot establish the full one-night setups in the candidate tests. These observations come from inspecting the routed form and booking list in `airbnb-clone-ts/src/pages/AddBooking.tsx` and `airbnb-clone-ts/src/pages/Bookings.tsx`.
