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

Only for products whose brand prints a note (`printed_gift_note_enabled`).
`printed_gift_note_mode`: `verbatim` (the card message), `auto_shortened`
or `custom` (both need `printed_gift_note_text`), or `disabled`. Leave it
out to keep the brand's usual note. A note that's too long is rejected
with the number of characters to cut; shorten it with the user.

## Question forms

`custom_form_id` attaches an existing form the recipient fills in when
accepting. These tools can't create forms, and forms are ignored for
`direct_send`.
