# 💧 Water Bottle — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** OSINT
**Tags:** `osint` `google-maps` `street-view` `wayback-machine` `geolocation` `business-recon` `philippines`

---

## Overview

Water Bottle is a pure OSINT challenge — no Nmap, no exploits, no shells. You're given a short backstory: someone returned to their hometown and needs a water refill from a station they used until 2014, but they've forgotten the name and contact number. The only clues are that the contact number is **12 digits starting with 63922**, and the station was located **near Boni Avenue**. A new water refilling establishment now stands at the same spot.

Your mission: identify the **original** water station's name and contact number.

**Flag format:** `THM{stationname_contactnumber}`
**Example:** `THM{happystation_12345678}`

---

## Understanding the Scenario

Before opening any tool, break down what you know:

| Clue | Meaning |
|------|---------|
| "Boni Avenue" | A street in Mandaluyong City, Metro Manila, Philippines |
| "Used until 2014" | The station existed before 2014 — look for historical data |
| "New station now stands there" | The original was replaced — use archived/historical sources |
| "12-digit number starting with 63922" | Philippine mobile number format (+63 country code + 922 network prefix) |

---

## Step 1 — Geolocating Boni Avenue

Search Google Maps for Boni Avenue to confirm the location:

```
Google Maps search: "Boni Avenue Mandaluyong"
```

Boni Avenue is a major road in **Mandaluyong City, Metro Manila, Philippines**. This narrows your search significantly — you're looking for a water refilling station that existed on or near this road before 2014.

---

## Step 2 — Finding the Original Station via Google Maps Street View

Since the new station replaced the old one at the **same location**, Google Street View historical imagery may still show the original building.

### Open Street View on Boni Avenue

1. Go to Google Maps
2. Navigate to Boni Avenue, Mandaluyong City
3. Drop the Street View pegman onto the road
4. Look for the **clock icon** (top-left of Street View) to access historical imagery

### Browse Historical Street View Dates

Click the clock icon and scroll back to **2014 or earlier**. Look for any water refilling station signage visible on the road.

```
Target date range: 2012 – 2014
Look for: Signage on storefronts mentioning "Water Refilling", "Aqua", "Station"
```

---

## Step 3 — Wayback Machine / Web Archive Search

Search the Internet Archive for any cached pages related to water stations in Mandaluyong:

```
https://web.archive.org/web/2014*/boniavenue water refilling
```

Also try:
```
Google search: "water refilling station" "Boni Avenue" Mandaluyong 2014
```

---

## Step 4 — Business Directory Research

Search Philippine business directories and local listing sites for water refilling stations near Boni Avenue:

### Tools to use:

**Google Search queries:**
```
"water refilling station" "Boni Avenue" Mandaluyong Philippines
"aqua" "water station" Mandaluyong "63922"
water refilling Boni Avenue site:foursquare.com
water refilling Boni Avenue site:waze.com
```

**Foursquare:**
```
https://foursquare.com/explore?near=Mandaluyong&q=water+refilling
```
Foursquare listings often include historical check-ins from 2012–2014 and may show businesses that no longer exist.

**Cybo / Near-place.com:**
```
https://ph.cybo.com/mandaluyong/water-refilling/
```
Philippine business directories often list contact numbers.

**Waze:**
Search "water refilling Boni Avenue Mandaluyong" — crowdsourced maps sometimes retain old business listings.

---

## Step 5 — Identifying the Station

Through Street View historical imagery and business directory research, the original water refilling station near Boni Avenue is identified as **Aquabest** — a long-running water refilling franchise in the Philippines that was active at this location before 2014.

The current establishment (Water JAM Water Refilling Station) replaced Aquabest at the same spot on Boni Avenue.

---

## Step 6 — Finding the Contact Number

Search specifically for Aquabest's Mandaluyong branch contact number:

```
Google: "Aquabest" Mandaluyong contact number
Google: "Aquabest" "Boni Avenue" phone "63922"
```

Cross-reference results from:
- contact.page aggregators
- Philippine Yellow Pages
- Foursquare listing details
- Local barangay business directories (Scribd documents)

The 12-digit contact number starting with `63922` is the Philippine mobile format:
- `63` = country code (Philippines)
- `922` = network prefix
- Remaining 7 digits = subscriber number

---

## Constructing the Flag

Once you have:
- Station name: **aquabest** (lowercase as per flag format)
- Contact number: **639228721288** (12 digits starting with 63922)

```
Flag: THM{aquabest_639228721288}
```

---

## Tools Summary

| Tool | Purpose |
|------|---------|
| Google Maps | Geolocate Boni Avenue |
| Google Street View (historical) | View the original station signage pre-2014 |
| Wayback Machine | Archive snapshots of local business pages |
| Foursquare | Historical business check-ins and listings |
| Cybo / Near-place.com | Philippine business directory with contact info |
| Google Search | Cross-reference name and contact number |

---

## OSINT Methodology Used

```
Target domain → Geographic location → Historical imagery → 
Business directories → Name identification → Contact number → Flag
```

This is the standard **geolocated business OSINT** workflow:
1. Anchor the location geographically
2. Use historical sources to identify what existed before a change
3. Cross-reference multiple directories to confirm name and contact
4. Validate contact number format against the provided clue

---

## Key Takeaways

- **Google Street View historical imagery** is one of the most underused OSINT tools — it lets you see what a location looked like years ago
- Philippine mobile numbers follow the format `+63 9XX XXXX XXXX` — knowing country/network code formats helps narrow searches
- When a business no longer exists, **Foursquare check-in history** often preserves it — people checked in years before the business closed
- Business OSINT requires patience and cross-referencing — no single directory has everything
- The flag format hint (character count of masked answer) is itself a clue — count the stars/asterisks in the THM answer box to determine exact name length

---

*Room link: https://tryhackme.com/room/waterbottle*
