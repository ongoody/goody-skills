# Sending from a saved template

1. `goody_templates_list` to find it (`include_catalog: true` adds
   Goody's own templates), then `goody_templates_get` to read it.
2. Pass `template_id` and the `version` you read as `template_version` to
   preview, price, and create. Use the same version for all three, and
   the same overrides for preview and create. Price accepts only some
   overrides (see the core flow), so pass it the template plus those.
3. The template fills in the cart, card, message, branding, and settings.
   Anything you pass explicitly overrides it.

What it doesn't do:

- It never picks recipients, the send method, or the payment method.
  Choose those as usual.
- It never counts as approval. Preview and get a yes like any other send.
- These tools can read templates but not create or edit them. To make
  one, the user saves it in the Goody web app.

If the template changed after you read it, the send is rejected. Read it
again, preview again, and get a new yes.
