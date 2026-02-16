# Newspack MerlinOne DAM Integration Plugin — Review

## What the spec gets right

- **Option A (AMF) over Option B (scratch)** is the correct call. The AMF framework is actively maintained (last release Nov 2025, 200K+ Packagist installs, used by NASA), and its Provider interface is genuinely simple — three methods.
- **Sideloading images (Option B for hosting)** is the right choice for a news org. Remote-URL attachments break `srcset` generation, and AMF itself has an open issue (#2) about responsive image srcsets not working with remote files. Sideloading sidesteps this entirely.
- **The plugin structure** is clean and conventional for WordPress.
- **Metadata mapping table** is well thought out, especially mapping credit/byline to `_image_credit` (which Newspack actually uses).

---

## Critical problems and questionable assertions

### 1. The API situation is far worse than the spec acknowledges

The spec says "MerlinOne's API docs are private — you need to request them" and treats this as a minor blocker. In reality:

- **Zero public documentation exists.** No OpenAPI spec, no endpoint references, nothing.
- **Zero open-source integrations exist** on GitHub. The only one (a Drupal module) was abandoned in Dec 2024 and required "MerlinOne server-side changes."
- **Canto acquired MerlinOne in August 2023.** The migration path for existing MerlinOne API customers is completely unclear. The old `merlinone.com/wordpress-connector` already redirects to Canto.
- The spec's assumed API interface (`search`, `get_asset`, `get_rendition_url`, `get_modified_since`) is **entirely speculative**. These endpoints may not exist in this form.

**This isn't "get the docs and go." This is "hope the API still exists and works the way you imagine."**

### 2. The spec ignores the Canto WordPress plugin entirely

Canto already ships an official WordPress plugin (`wordpress.org/plugins/canto/`, v3.1.1, June 2025) that:

- Supports Gutenberg blocks, Classic Editor, ACF, and Elementor
- Has global search across Canto DAM
- Does automatic duplicate checking
- Runs background updates via WP-Cron
- Is actively maintained (3 contributors, security patches applied)

If BDN's MerlinOne instance is being migrated to Canto (which is likely post-acquisition), **the existing Canto plugin might solve the problem with zero custom development.** The spec doesn't mention this at all.

### 3. AMF is in alpha

The spec presents AMF as a battle-tested foundation. It's labeled "alpha" by its own maintainers. The responsive image srcset issue (#2) is still open and directly affects news sites that need multiple image sizes. With sideloading (Option B) this is mitigated, but it's worth noting.

### 4. WP-Cron for 15-minute sync is unreliable

The spec acknowledges "or a real cron if available" in passing, but WP-Cron only fires on page loads. On a low-traffic staging site or during off-hours, a 15-minute interval could stretch to hours. For a news org where photo timeliness matters, this needs a real cron job or, better, webhooks. **Canto supports 14 webhook event types** including `New Asset` — if the MerlinOne instance migrates to Canto, push-based sync is available.

### 5. "Stored encrypted in wp_options" for the API key is hand-waving

WordPress doesn't natively encrypt option values. You'd need to implement encryption yourself (using `sodium_crypto_secretbox` or similar) and manage the key. More commonly, plugins just store API keys in plaintext in `wp_options` or use environment variables / `wp-config.php` constants. The spec should specify which approach.

### 6. The dual-purpose design is potentially redundant

The plugin both:

- Provides an AMF media modal (browse MerlinOne from the editor)
- Runs background sync (import everything into WP media library)

If you're sideloading everything via sync, editors can already find those images in the normal media library. The AMF modal becomes useful only for images not yet synced. This overlap should be explicitly designed for — does the modal search MerlinOne *and* local? Just remote? How do you avoid duplicates when an editor selects an image that sync will also import?

---

## What I'd suggest instead

**Step 0: Determine whether BDN's MerlinOne is migrating to Canto.** If yes, evaluate the existing Canto plugin first. It may need only minor customization (metadata mapping, Newspack credit field) rather than a ground-up build.

**If custom development is still needed:**

1. **Get the API docs before writing a spec.** The current spec is speculative architecture. Write the spec *after* you know what endpoints exist, what auth looks like, and whether "modified since" queries are supported.
2. **Choose one sync model, not two.** Either:
   - **AMF-only (on-demand):** Editor searches MerlinOne from the modal, selects an image, it sideloads at that moment. No background sync. Simpler, no cron dependency.
   - **Full sync + native media library:** Background job imports everything. No AMF modal needed — editors just use the regular media library. Better for large newsrooms where photos should "just be there."
3. **Use webhooks if available** instead of polling. Canto supports them; MerlinOne might too.
4. **Use `wp-config.php` constants for credentials** (`MERLINONE_API_KEY`), not encrypted wp_options. This is the WordPress convention for secrets.

---

## Scale and resource estimate

| Component | Complexity | Estimated effort |
|---|---|---|
| API client class | Medium (depends entirely on API docs) | 2–3 days |
| AMF Provider | Low (3 methods, straightforward mapping) | 1 day |
| Background sync with dedup | Medium-High (edge cases: failures, partial syncs, metadata updates) | 3–5 days |
| Settings page | Low | 1 day |
| Testing on Newspack | Medium (compatibility, block editor, featured images) | 2–3 days |
| Webhook handler (if supported) | Low-Medium | 1–2 days |
| **Total dev time** | | **~2–3 weeks** for one experienced WP developer |

### Blockers that add unknown time

- Getting API documentation (days to weeks of back-and-forth with Canto)
- Discovering API doesn't support expected operations (could require redesign)
- AMF alpha quirks in production Newspack environment

---

## How much can Claude do?

**Claude can do most of the code, but can't unblock the real bottleneck.**

### What Claude can build right now

- Complete plugin scaffold (bootstrap, settings page, class structure)
- AMF Provider implementation (once you provide the API response format)
- Background sync logic with WP-Cron scheduling, dedup, and metadata mapping
- API client class (with placeholder endpoints that you fill in from docs)
- Unit test stubs

### What Claude cannot do

- **Get you the API documentation** — that requires a human emailing Canto
- **Test against the live MerlinOne instance** — needs real credentials and network access
- **Verify AMF compatibility** with your specific Newspack version — needs a running WordPress environment
- **Make the strategic decision** about whether to use the existing Canto plugin instead

---

## Bottom line

If you get the API docs and confirm the endpoints, Claude could generate ~80% of the plugin code in one session. The remaining 20% is integration testing, edge case handling against real API responses, and Newspack-specific tweaks that require a live environment. The spec is architecturally sound but built on an unverified foundation — **the first thing to do is not write code, it's make a phone call to Canto.**
