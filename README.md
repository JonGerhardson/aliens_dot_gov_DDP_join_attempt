Work in progress, and having some issues uploading to github right now. Placeholder AI generated summary of project below. For map data see [https://github.com/Jongerhardson/motherfuckers](https://github.com/Jongerhardson/motherfuckers)

 The White House published an interactive map on aliens.gov showing immigration "arrests" as thousands of dots
  across the country. After WIRED reported on it, the White House quietly scrubbed it — cutting the totals roughly
  in half and removing dots that named US citizens. We'd already recovered the original, pre-scrub version from
  the Wayback Machine (captured May 29): 11,873 location dots, 476,547 arrests.

  Separately, we have the Deportation Data Project (DDP) files — the real ICE enforcement records (~713,000
  arrests, plus detainers and detention histories), obtained through FOIA. This is ground truth.

  The whole session was about one question: can we connect the White House's map to the real ICE records, and what
  does that connection tell us?

  The central idea, and the first real decision

  The map is aggregated — each dot is a town, and it only tells you how many were arrested there, which countries
  they came from (as a list), which charges (as a list), and a date range. That's useless for matching
  individuals… except when a dot represents exactly one arrest. Then the list collapses to a single country, a
  single exact date, a single place.

  Decision: focus on the ~4,300 single-arrest dots. Everything else can't be tied to a person. That's where all
  the leverage is.

  To match a dot to an ICE record, we need fields both sides share. ICE records have a state, a date, and a 
  citizenship country — but no city. So the join key became state + date + country. Match a dot, and you've turned
  an anonymous dot into a specific ICE person, which unlocks their whole detention history.

  About 800 dots matched exactly one ICE person. The rest matched 2–5 or many (common nationalities on busy days),
  or nothing.

  Four angles, and what each was for

  We pursued four ways to get more out of this:

  - D — Geography. The hope: ICE records have location hints (office codes, landmarks) that could break ties when
  several people match the same state/date/country. What we learned: ICE codes each arrest to a regional office,
  not the actual town — so location is ~130 km fuzzy and mostly couldn't pin individuals. Only when a landmark
  literally spells out the town ("Firebaugh Police Department") is it precise. Decision: use geography only where
  it's exact, not as a blunt filter.
  - F+G — Charges and gangs. The White House lists charge categories; ICE lists conviction charges with the same
  coding system. The hope: use charges to break ties. What we learned: they agree only ~76% of the time (the map
  shows the arrest charge; ICE records the most serious conviction — different things). Decision: treat charges as
  corroboration, never as proof.
  - L — US citizens. The map named 715 dots with US-born arrestees. ICE's enforcement data is, by definition,
  non-citizens only. What we found: there are zero US citizens anywhere in the ICE arrests, detainers, or
  detention data. So those 715 dots cannot be ICE deportation arrests — strong evidence the map mixed in something
  else (non-immigration arrests).
  - I — The big-picture comparison. Instead of person-by-person, compare totals. What we found: the pre-scrub
  map's state-by-state numbers track ICE's real arrest numbers almost perfectly (correlation 0.996), at a flat
  ~1.3× ratio everywhere — which is just the map covering ~3 more months than the ICE data. Meaning: the original
  map was an honest reflection of ICE arrests. The scrub then cut it nearly in half, pushing it below documented
  reality — which contradicts the White House's claim that it was only removing "non-immigration" arrests.

  The most important findings (L and I) don't even depend on the fragile person-matching.

  The discipline that ran through everything

  The recurring decision was: never trust a matching signal until it's validated against known-good matches. Every
  shortcut — using the office region, using charges, using county — was tested against the ~800 confirmed matches
  first. Each time, if it was too unreliable (region was 67% right, charges 76%, county 67%), we refused to use 
  it to assert a match and instead kept it as a clearly-labeled, lower-confidence tier. That's why the final
  crosswalk is tiered by confidence rather than a single flat list.

  The skepticism test

  You asked the right hard question: what if the map is fabricated or noise-injected, and the matches are just 
  coincidences that happen to fit?

  We tested it directly: shuffle the dates and re-run the matching 2,000 times. If the map were fake, the real
  dates would match no better than random shuffles. Result: the real matches beat random by ~50 standard
  deviations. The map is unambiguously real data.

  But the test also revealed the honest caveat: random shuffling still produces ~440 "unique" matches by chance,
  because rare combinations (one Angolan in Maine) match easily either way. So we computed a per-dot coincidence 
  probability and labeled each match accordingly — rare-nationality matches are bedrock; "Mexico in Texas" matches
  are coincidence-prone. About 200 are essentially certain; ~640 of 800 are genuine signal.

  The review, and the fixes you prompted

  A correctness pass (and your sharp catches) surfaced three real issues:

  1. Territories — Guam/Virgin Islands dots were being silently dropped. Fixed (+15).
  2. County matching — added to break more ties, but validation showed it's only ~67% reliable, so we kept it in a
  separate, clearly-weaker tier rather than mixing it with the solid matches.
  3. Wrong state labels — your "LaGrange, IL is in Cook County" catch exposed that ~333 dots (7.7%) have a typo'd 
  state in the text (the coordinates are right, the label is wrong: "Des Plaines, IA" is really in Illinois).
  These had been matched against the wrong state's ICE records. Decision: trust the coordinates, not the text
  label, for state. This removed dozens of bogus matches — the count went down, which is the correct direction.

  Then two more judgment calls:
  - The contested flag (done): when two dots both claim the same single ICE person (physically impossible — at
  most one is right), flag both for review instead of silently trusting them. Five such collisions exist; now
  they're visible.
  - A "border-aware" state rule (rejected): it would have fixed one border-town misflip (Conklin, NY) but
  re-broken ~5 legitimate typo corrections. Decision: not worth it — the contested flag already surfaces the one
  bad case.

  Where it landed

  A confidence-tiered crosswalk of ~1,180 White House dots tied to specific ICE individuals, honestly labeled from
  "near-certain" down to "lead only," with coincidence-prone and contested cases flagged. And, more durably,
  three conclusions that don't rely on any single match:

  1. The map is real ICE data, not fabricated or synthetic.
  2. The scrubbed version undercounts real ICE arrests by ~46%, contradicting the stated reason for the scrub.
  3. The map included US citizens who provably aren't in ICE's enforcement records at all.

  The throughline in every decision: prefer the conclusion the data can actually support, validate before 
  trusting, and label uncertainty honestly rather than inflate the match count.
