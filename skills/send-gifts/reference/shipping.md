# Shipping and international recipients

## Before promising a gift can arrive

Neither preview nor price checks that a product can reach an address.
Check `goody_products_get` → `shipping_tier` and `restrictions` first,
and pass `shipping_country` when searching.

## Shipping a physical item (`direct_send`)

- Every recipient needs a full `mailing_address`. Ask for it; never
  invent one. `state` and `postal_code` only where the country uses them.
- Works in supported countries, not only the US.
- Can't be used with the single `contact_id` shorthand: contacts store no
  address. Use inline `recipients`, each with its own address.
- No email goes out, and meeting requests aren't allowed.

## Recipients outside the US or Canada

Ask about it whenever a recipient might be abroad:

- `international_shipping_tier: "full"` reaches 140+ countries, with up
  to $55 per order in taxes, tariffs, and freight. The preview may not
  show that fee, so mention it.
- `international_gift_cards_enabled: true` lets someone whose country
  can't get the item swap for a gift card.

A US or Canada recipient, or a digital gift, needs none of this.
