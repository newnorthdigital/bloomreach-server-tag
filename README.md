# Bloomreach Discovery Server Pixel GTM Tag

**A server-side Google Tag Manager template that sends Bloomreach Discovery pixels from your sGTM container through Bloomreach's pixel API.**

[![Created by Freek Kampen](https://img.shields.io/badge/Created%20by-Freek%20Kampen-455CE9)](https://freekkampen.com) [![Maintained by New North Digital](https://img.shields.io/badge/Maintained%20by-New%20North%20Digital-455CE9)](https://newnorth.nl/?utm_source=github&utm_medium=gtm-template&utm_campaign=bloomreach-server-tag)

## Features

- Sends page views (all Bloomreach page types, including conversion with basket), virtual page views, add to cart, search submit, suggest click and quick view events to `https://p.brsrvr.com/pix.gif`.
- Adds the parameters Bloomreach requires for server-side pixels: `version` (prefixed `ss-`), `cookie2`, `client_ip`, `client_ts` in microseconds, `rand`, `url`, `ref`, `type` and the visitor's `user-agent` header.
- Manages the `_br_uid_2` visitor cookie server-side: reuses the existing cookie so current visitors keep their history, creates one for new visitors, and increases the hit count on every hit. Because the cookie is set by your server, Safari's 7-day cap on script-set cookies does not apply.
- Reads `br_<parameter>` event parameters when a field is empty: `br_ptype`, `br_prod_id`, `br_prod_name`, `br_sku`, `br_cat_id`, `br_cat`, `br_search_term`, `br_item_id`, `br_item_name`, `br_catalogs`, `br_title`, `br_user_id`, `br_q` and `br_aq`. No Event Data variables needed.
- Page types are trimmed and lowercased. Values Bloomreach does not accept are sent as `other` and logged as an error. In GTM Preview, `debug=true` is added automatically.
- Otherwise, defaults every field to GA4 event data: `page_location`, `page_referrer`, `page_title`, `language`, `items`, `transaction_id`, `value`, `currency`, `search_term`. Every field can be overridden.
- Skips hits where GA4 reports `analytics_storage` as denied (`gcs`), and skips user agents on Bloomreach's blocklist (curl, wget, python-requests, bots, crawlers and similar).
- Errors are always logged, also in production. Per-hit logging is optional.

## Known limits

- `client_ts` is the time sGTM receives the hit, not the browser time. GA4 does not forward a client timestamp.
- The bot filter matches `bot` only when followed by `/ ; ) - +` or at the end of the user agent, so phone models like CUBOT get through. A few bots with other formats (Slackbot 1.0, Pingdom) are not filtered; they do not run GA4 anyway.
- Several tags firing on the same incoming request read the same cookie, so they send the same hit count (`hc`).

## Web or server?

There is also a [web version](https://github.com/newnorthdigital/bloomreach-web-tag). Use one or the other per site, never both: Bloomreach warns that sending the same events client-side and server-side for more than a few hours corrupts analytics and search performance. When migrating, test with debug events first, then switch server-side live and remove the client-side pixel right away.

## Requirements

- An sGTM container on a subdomain of your site (for example `sst.example.com`), so `_br_uid_2` is a first-party cookie on your domain.
- GA4 events arriving through the GA4 client. The browser tag must send the data Bloomreach needs, such as a page type, ecommerce items and search terms.

## Installation

### From the Community Template Gallery
1. In a GTM server container, open **Templates → Tag Templates → Search Gallery**.
2. Search for **Bloomreach Discovery Server Pixel by New North** and add it.

### Manual installation
1. Download `template.tpl` from this repo.
2. In GTM: **Templates → New → ⋮ → Import**, select the file, and save.

## Setup guide

1. **Page view tag.** Pixel type **Page view**, your **Account ID**, and **Page type** on **Automatic**. A GA4 `page_view` carries no product or category data, so send what Bloomreach needs as `page_view` event parameters from the browser, named after the Bloomreach parameter with a `br_` prefix: `br_ptype`, `br_prod_id`, `br_prod_name`, `br_sku`, `br_cat_id`, `br_cat`, `br_search_term`, `br_item_id`, `br_item_name`, `br_catalogs`. The tag reads them without any Event Data variables. With a fixed page type instead of Automatic, the fields for that page type appear, and a value there wins over the event parameter. Trigger: GA4 `page_view`, excluding the order confirmation page.
2. **Conversion.** A second Page view tag with page type `conversion`, triggered on the GA4 `purchase` event. Order ID, value, currency and basket come from the purchase event. Exclude the confirmation page from the tag in step 1, or Bloomreach receives two page views for it.
3. **Events.** One tag per event type: **Add to cart** on `add_to_cart`, **Search submit** on `search`, **Suggest click** and **Quick view** on your own events.
4. **Validate.** In GTM Preview the tag sends debug events on its own; check Event diagnostics in Integration mode. Tick **Send as debug events** only to test outside Preview. Bloomreach discards events with problems in `version`, `client_ip`, `client_ts` or `user-agent`.

## Field reference

| Field | Bloomreach parameter | Default |
|---|---|---|
| Account ID | `acct_id` | Required. |
| Domain key, view ID, user ID | `domain_key`, `view_id`, `user_id` | Only when your account needs them. |
| Page type | `ptype` | Automatic: `br_ptype`, else `other`. Accepts a variable. |
| Page title | `title` | `page_title` |
| Product ID / name / SKU | `prod_id`, `prod_name`, `sku` | First item in `items` |
| Category ID / name | `cat_id`, `cat` | |
| Search term | `search_term` | `search_term` |
| Content item ID / name | `item_id`, `item_name` | |
| Order ID, basket value, currency | `order_id`, `basket_value`, `currency` | `transaction_id`, `value`, `currency` |
| Basket items | `basket` | `items`, in Bloomreach's `!i<id>'s<sku>'n<name>'q<qty>'p<price>` format. `!` and `'` inside names are escaped. |
| Search query / typed query | `q`, `aq` | `search_term` |
| Catalogs | `catalogs` | Comma-separated names, sent as `cat0=…!cat1=…` |
| GA4 item ID mapping | | Whether GA4 `item_id` is the product ID or the SKU. |
| Additional parameters | any | E.g. Relevance by Segment fields. |
| Send as debug events | `debug=true` | |
| Mark as test data | `test_data=true` | |
| Skip hits where analytics_storage is denied | | On |
| Page URL / referrer override | `url`, `ref` | `page_location`, `page_referrer` |

## Permissions

- Reads event data and request headers (user agent, client IP).
- Reads and sets the `_br_uid_2` cookie.
- Reads container data (to detect GTM Preview).
- Sends HTTP requests to `https://p.brsrvr.com/*`.
- Logs to the console in all environments (errors always, per-hit logs only when enabled).

## Resources

- [Server-side pixel integration](https://documentation.bloomreach.com/discovery/docs/server-side-pixel-integration)
- [Implementation guide (server-side pixels)](https://documentation.bloomreach.com/discovery/docs/implementation-guide-server-side-pixels)
- [Migrating from client-side to server-side tracking](https://documentation.bloomreach.com/discovery/docs/migrating-from-client-side-to-server-side-tracking)
- [Pixel parameter reference](https://documentation.bloomreach.com/discovery/docs/pixel-reference)

## Author

Created and maintained by [Freek Kampen](https://freekkampen.com) at [New North Digital](https://newnorth.nl/?utm_source=github&utm_medium=gtm-template&utm_campaign=bloomreach-server-tag).

## License

Apache 2.0, see [LICENSE](LICENSE).
