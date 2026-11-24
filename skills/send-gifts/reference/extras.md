# Meeting requests, printed notes, and question forms

## Meeting requests

`meeting_setting` asks the recipient to book time: `gift_first` (accept,
then asked to book) or `meeting_required` (must book first). Use it
instead of pasting a Calendly link into the message.

- It needs a connected Calendly link
  (`meeting_scheduling.calendar_connected`). If there isn't one, send the
  user to Goody's web settings.
- If the user didn't mention a meeting, follow
  `meeting_scheduling.default_setting` and say so in the confirmation.
- If that default gates a gift that isn't prospecting (a thank-you, a
  candidate, a renewal), point it out and ask. Being made to book a
  meeting before claiming a thank-you reads badly.
- Pass the same value to preview and create. Not allowed with
  `direct_send`.

## Printed notes in the box

Don't set `printed_gift_note_mode` or `printed_gift_note_text` from here.
Some brands charge a fee per printed note, and preview and price don't
accept those fields, so the fee would be missing from the total the user
approved. Leave them out; the brand's usual note still prints. If the
user wants a custom printed note, point them to the Goody web app.

## Question forms

`custom_form_id` attaches an existing form the recipient fills in when
accepting. These tools can't create forms, and forms are ignored for
`direct_send`.
