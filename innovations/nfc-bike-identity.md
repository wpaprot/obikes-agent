# NFC Bike Identity — my.obikes.cc/{SN}

> Topic: Embedding NFC tags in O-Bikes frames to link each bike to a digital identity page.
> Started: 2026-04-01

---

## The Concept

Every O-Bikes frame ships with a permanently embedded NFC tag. Scanning it opens:

```
https://my.obikes.cc/{bike-SN}
```

The page is **public by default** (basic info visible to anyone), but shows **more to authenticated owners**.

This turns each bike into a connected object — a digital passport that lives with the frame forever.

---

## What the URL Shows

### Public (unauthenticated visitor)
- Model name and year (e.g. O|1 2025)
- Frame color / build spec (if not custom/private)
- "This bike is registered to an O-Bikes owner" — no personal data
- O-Bikes brand story, link to shop
- Option to report a stolen bike (flag it)
- If bike is marked stolen: **big red banner** — "This bike has been reported stolen"

### Authenticated — Owner
- Full build spec (groupset, wheels, all components)
- Frame serial number + manufacturing date
- Service history log (owner can add entries)
- Ride stats if connected to Strava/Garmin (optional integration)
- Warranty status
- Transfer of ownership flow (sell the bike → new owner claims it)
- Custom notes / fit data storage

### Authenticated — O-Bikes Admin
- Everything above
- Manufacturing batch, supplier, paint lot
- QC notes from production
- Any warranty claims

---

## Technical Architecture

### NFC Tag Selection

| Requirement | Decision |
|---|---|
| Chip | **NTAG424 DNA** — AES-128 crypto auth, each tap generates signed token backend can verify |
| vs NTAG216 | NTAG216 only returns a URL — anyone can clone it with a sticker. NTAG424 is unforgeable |
| Form factor | **Hard disc tag, 16mm, PPS housing** — rigid, won't crumple during install, handles -40°C to +85°C |
| vs wet/dry inlay | Inlays are 0.07mm PET, fragile, designed for label lamination not direct bonding inside a tube |
| Memory | 424 bytes on NTAG424 — enough for URL + SN payload |
| Supplier | **Seritag** (seritag.com) — small MOQ, ships to EU, responsive |

**Key decision: where to embed the tag?**

Options:
1. **Inside the headtube** — protected, always accessible without removing anything, survives crashes
2. **Under the bottom bracket** — less aesthetically intrusive but harder to scan
3. **Inside the top tube** — clean but requires specific phone placement
4. **On a removable top cap** (stem cap) — easy to swap, but loses the "inseparable from frame" property

**Recommendation: headtube interior.** It's the most scannable spot (thin carbon wall, NFC penetrates well), protected from impact, and a natural place to look for a bike's identity. Some high-end brands (Specialized S-Works) use this approach.

### Tag Embedding Process

- Tag is adhered to the inside of the headtube before fork/headset install
- Covered with a thin layer of epoxy or frame liner for protection
- NFC signal penetrates carbon fiber fine (not metal — avoid aluminum frames without a window)
- For aluminum frames: embed in a small recess with a non-conductive cover or use headtube liner

### URL Encoding on Tag

NDEF record, type URI:
```
https://my.obikes.cc/O1-2025-00042
```

Serial number format suggestion:
```
{model}-{year}-{5-digit-sequence}
  O1    -2025  -00042
  OX    -2025  -00001
  OG    -2026  -00007
```

---

## Web Platform — my.obikes.cc

### Stack considerations (you're IT people, so blunt):

- Static public pages with SSR for SEO — Next.js or SvelteKit
- Auth: simple email magic link or Google OAuth — no passwords to manage
- DB: Postgres, bike ownership as a simple table (serial → owner_id)
- Ownership claim: owner enters frame SN + purchase email → verified → access granted
- Could be a subdomain of main site or standalone

### Ownership Claim Flow

```
1. New owner scans NFC → lands on my.obikes.cc/SN
2. Page shows: "This bike is unregistered" or "Registered to previous owner"
3. Button: "Claim this bike"
4. Enter email used at purchase (or dealer code if sold through shop)
5. Magic link sent → confirmed → bike linked to account
6. Transfer: current owner initiates transfer → new owner claims
```

### Stolen Bike Registry

This is a **real differentiator**. If a bike is reported stolen:
- SN flagged in DB
- Public page shows red stolen banner
- Anyone who scans the tag (e.g. a buyer at a flea market) sees it immediately
- Optional: integrate with Bike Register (UK), or velopass.org for wider reach

---

## Business Value

1. **Anti-theft / recovery** — buyers can verify a bike isn't stolen before purchasing used
2. **Brand premium** — NFC tag signals high-tech, thoughtful design. Cannondale, Trek don't do this yet at this level
3. **Customer relationship** — you know who owns each bike, can reach them for recalls, service reminders, new product launches
4. **Resale market** — ownership history follows the frame; O-Bikes stays in the loop on secondhand sales
5. **Service network** — a local bike shop scans the tag and sees the full build spec instantly
6. **Content hook** — "scan to see your bike's story" is genuinely shareable, Instagram-worthy at delivery

---

## Open Questions / Next Discussion Points

- [ ] Do we embed the tag during manufacturing (supplier-side) or in-house before delivery?
- [ ] What data is mandatory on the public page vs opt-in by owner?
- [ ] Do we integrate with Strava/Wahoo/Garmin from day one or leave as v2?
- [ ] Stolen bike: do we notify law enforcement or just flag for community?
- [ ] my.obikes.cc — standalone product or part of main obikes.cc?
- [x] Tag supplier: **Seritag**, 16mm disc, NTAG424 DNA, PPS housing
- [x] Adhesive: **Araldite Standard** (2-part epoxy), applied in-house before headset press
- [x] Form factor: hard disc tag, not wet/dry inlay
- [x] Embedding: done by O-Bikes in-house, not by frame manufacturer
- [ ] Do custom bikes get the same SN scheme or separate?

---

## Tag Purchasing — What to Buy

### For carbon fiber frames (O|1, O|X, O|G)

**Buy: Seritag 16mm Ultrathin Disc, NTAG424 DNA, PPS housing**

- seritag.com → Disc Tags → filter NTAG424, PPS material
- 16mm diameter, 1.3mm thick — fits inside any headtube
- Hard-cased (won't crumple during installation)
- PPS housing: rated -40°C to +85°C (handles Spanish summer + Polish winter)
- NTAG424 DNA chip: AES-128 cryptographic authentication — each tap generates a unique signed token, so the backend can verify "this tap came from a real O-Bikes chip, not a clone"
- This is better than NTAG216 for a brand authenticity use case

**Start here first:** Order the [Disc Tag Sample Pack](https://seritag.com/nfc-tags/disc-tag-pack) — €28.45 for 16 different disc tags. Test read-through on your actual frame prototype before committing to a batch.

**Pricing at small volume (email seritag.com for exact quote):**
~€1.20–€2.00/tag for NTAG424 at 100 units. On a premium bike, irrelevant.

### For aluminum frames (if you ever do them)

Standard tags won't work inside an aluminum tube — aluminum is a Faraday cage at NFC frequencies.
Use: **Seritag 29mm On-Metal NTAG424** placed on outer surface under a decal.

---

## Physical Installation

### Where: headtube interior (flat against inner wall)

Position the tag so the antenna plane is **parallel to the tube wall** — the NFC coil must face outward through the carbon, not along the tube axis. A phone pressed to the outside of the headtube will couple through 2-4mm of carbon at 1-2cm range.

### Adhesive decision — context

Frame manufacturer will not embed the tag during production. Installation is done by O-Bikes in-house, before headset is pressed. This means the adhesive must:
- Bond reliably to the interior of a carbon fiber headtube (curved surface, possibly dusty/release-coated)
- Survive the full bike lifetime without peeling — this is permanent, not serviceable
- Handle temperature range: -25°C to +75°C operating (Polish winter → Spanish summer, bike in a hot car)
- Not be a tape that can be questioned for long-term reliability

**Wet inlay and dry inlay are not suitable** — too fragile (0.07mm PET substrate), will crumple during installation inside a tube, and adhesive on wet inlay is a lamination adhesive not a bonding adhesive.

---

### Recommended adhesive: Araldite Standard (2-part epoxy)

**Why:**
- Epoxy does NOT attenuate NFC signal — electrically non-conductive, magnetic field passes through freely
- Araldite Standard is a structural adhesive, bonds carbon fiber excellently
- Slow cure (90 min workable, 24h full strength) — no exotherm risk at small volume
- The exotherm concern (80-120°C spike) only applies to large epoxy masses (filling a mold). A pea-sized dab bonding a 16mm disc generates negligible heat — not a real concern here
- Rated to 80°C+ when cured, rigid, permanent
- Available everywhere in Poland/EU, cheap

**Alternative: Sikaflex 552 (polyurethane adhesive/sealant)**
- Slightly flexible when cured — handles vibration fatigue better over years of riding
- Excellent adhesion to composites, used in marine and automotive bonding
- Waterproof, rated -40°C to +90°C
- Choose this if concerned about micro-flex of the carbon frame cracking a rigid epoxy bond over time
- Slower cure than epoxy, messier to work with

**3M VHB 4950 tape — considered, not preferred**
- Technically rated to 93°C, high-tack
- Rejected: tape adhesion on a curved interior surface inside a tube is harder to execute reliably than liquid adhesive; long-term peel risk on a curved bond; founders not confident in permanent tape for a lifetime bond

**Decision: use Araldite Standard for early production. Revisit Sikaflex if any bond failures observed.**

---

### Installation steps (DIY, pre-headset)

1. Roughen the back of the disc tag with 120 grit sandpaper (10 seconds)
2. Clean headtube interior at target spot with IPA, let dry fully
3. Mix small amount of Araldite Standard — pea-sized, no more
4. Apply to back of tag
5. Use a long thin tool (zip tie, wooden skewer, long tweezers) to position and press flat against inner wall, **antenna facing outward through the carbon wall**
6. Hold or wedge for 10-15 min until it won't slide
7. Leave 24h before pressing headset
8. Encode the NDEF URL using **NFC Tools Pro** (Android) or Seritag's own app

**Critical orientation:** antenna plane must be parallel to the tube wall — NFC coil faces outward. If mounted axially (coil pointing along the tube), it will not read from outside.

---

## Tag Suppliers

| Supplier | Notes |
|---|---|
| **Seritag** (UK, ships EU) | Best for early stage — small MOQ, responsive, good docs |
| **ShopNFC** | Good for NTAG216 wet inlays if you want thinner/cheaper option |
| **ZipNFC** | UK, similar to ShopNFC, competitive pricing |
| **NXP directly** | For volume (10k+), not relevant now |

Note: "TagMatic" doesn't exist as a tag vendor — that was a bad suggestion in the initial notes. Ignore it.

---

## Related Ideas (parking lot)

- QR code backup printed on frame (NFC won't work with all phones without NFC)
- NFC on packaging — scan box to register warranty before even opening it
- "Bike passport" PDF generated from the page, downloadable for insurance purposes
- Integration with cycling insurance providers (Velosurance, Laka)
