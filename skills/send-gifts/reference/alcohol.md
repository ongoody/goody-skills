# Alcohol gifts

Check all three before building the gift, or the send is rejected after
the user has already chosen everything:

1. **The account must have alcohol turned on.** If the error says it
   isn't, tell the user to contact Goody support. Don't retry.
2. **Pay with a card.** Balance can't pay for alcohol.
3. **Age check.** Pass `alcohol_age_verification_attested: true` only
   after the user confirms they're over 21. Never attest for them.

Search leaves alcohol out unless `exclude_alcohol` is false. Only set
that when the user asked for alcohol.
