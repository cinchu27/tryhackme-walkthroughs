# ✉️ Letter — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** OSINT / Historical Research
**Tags:** `osint` `historical-research` `image-analysis` `maritime-history` `french-history` `document-analysis`

---

## Overview

Letter is a refreshing departure from typical CTF rooms — there's no Nmap, no exploit, no shell. Instead, you're handed a **scanned old envelope**, a **torn newspaper clipping**, and a **handwritten note**, then asked to piece together the story behind them. The challenge demands careful observation, historical research, and methodical OSINT across French maritime records and historical archives.

If you enjoy document forensics and historical investigation over traditional hacking, this room is for you.

---

## Understanding the Artifacts

The room presents three key artifacts:

### Artifact 1 — The Envelope
- Addressed to someone named **"Edouard G."**
- Sent to an **SNSM station**
- The envelope is **torn** — the full address is not visible
- **SNSM** = *Société Nationale de Sauvetage en Mer* (French National Sea Rescue Organisation)

### Artifact 2 — The Newspaper Clipping
- From **L'Ouest-Éclair** — a historical French regional newspaper
- Dated **"Jeudi 28 Mai"** (Thursday, May 28)
- Describes a **maritime disaster** in the **Finistère region** of France
- The clipping is damaged — some text is unreadable

### Artifact 3 — The Handwritten Note
- Contains partial text referencing the disaster and the recipient
- Provides additional context clues for identifying the event and location

---

## Step 1 — Identifying the Newspaper

**L'Ouest-Éclair** was a major French regional daily newspaper published in Rennes, Brittany. It ran from 1899 to 1944.

```
Google search: "L'Ouest-Éclair" newspaper history France
```

This confirms the letter dates from somewhere between **1899 and 1944** — a significant timeframe for maritime disasters in Brittany.

---

## Step 2 — Finding "Jeudi 28 Mai" (Thursday May 28)

The clipping is dated Thursday, May 28. To find which year May 28 fell on a Thursday, use a perpetual calendar:

```
Google search: "May 28 Thursday" year 1899 1944
```

Or use an online calendar tool:
```
https://www.timeanddate.com/calendar/
```

Cross-reference with known maritime disasters in Finistère during those years to narrow down the exact date.

---

## Step 3 — Researching the Maritime Disaster

With the newspaper name, the date range, and the Finistère region, search historical maritime disaster records:

```
Google search: maritime disaster Finistère "28 mai" France historical
Google search: "L'Ouest-Éclair" "28 mai" naufrage Finistère
```

**French search terms to use:**
- `naufrage` = shipwreck
- `catastrophe maritime` = maritime disaster
- `sauvetage en mer` = sea rescue
- `Finistère` = the westernmost department of Brittany

Search **Gallica** (France's national digital library — BnF), which has digitised historical French newspapers including L'Ouest-Éclair:

```
https://gallica.bnf.fr/
Search: L'Ouest-Éclair + date range + naufrage Finistère
```

---

## Step 4 — Identifying "Edouard G." at the SNSM Station

Once you've identified the maritime event, research the SNSM station connected to it:

```
Google search: SNSM station Finistère "[disaster location]" history
Google search: "Edouard" SNSM Finistère maritime rescue
```

**SNSM historical records** can be found via:
- The SNSM official website: `https://www.snsm.org/`
- Regional maritime heritage archives
- Local Breton historical societies

Look for a person named **Edouard** with surname beginning with **G** connected to the identified SNSM station and the disaster event.

---

## Step 5 — Confirming the Full Address

With the SNSM station identified and the person named, you can reconstruct the torn envelope address:

```
[Edouard G. full name]
Station SNSM de [Location]
[Town], Finistère, France
```

Cross-reference with the newspaper clipping location and any historical crew/volunteer lists for the station.

---

## Step 6 — Answering the Room Questions

The room asks a series of questions based on your investigation. Work through them in order:

1. **What organisation does the envelope reference?** → SNSM (Société Nationale de Sauvetage en Mer)
2. **What region/department is referenced in the newspaper?** → Finistère
3. **What was the date of the disaster?** → Derived from perpetual calendar + historical records
4. **Who is the letter addressed to?** → Edouard G. (full name from maritime records)
5. **What SNSM station was it sent to?** → Identified from disaster location research

---

## Tools & Resources Used

| Tool | Purpose |
|------|---------|
| Google Search (French terms) | Historical maritime disaster research |
| Gallica (gallica.bnf.fr) | Digitised L'Ouest-Éclair newspaper archives |
| Perpetual Calendar (timeanddate.com) | Match "Thursday May 28" to a specific year |
| SNSM.org | French sea rescue organisation history |
| Wikipedia (French) | Maritime disasters in Brittany / Finistère |
| Google Translate | Translating French historical documents |

---

## OSINT Methodology Used

```
Artifact analysis → Organisation identification → 
Date narrowing (perpetual calendar) → Historical newspaper archive → 
Maritime disaster records → Person identification → Flag
```

This is a **historical document OSINT** workflow:
1. Extract all visible clues from the artifact
2. Identify institutions and publications referenced
3. Use archives and databases to find the specific event
4. Cross-reference multiple sources to confirm identity

---

## Key Takeaways

- **Document artifacts** (envelopes, newspapers, handwritten notes) are rich OSINT sources — every visible detail is a clue
- **Gallica** (BnF) is an invaluable resource for French historical documents — millions of digitised pages available for free
- A **perpetual calendar** turns a "Thursday May 28" into a specific year — always identify the exact date when given day + date combinations
- **Searching in the language of the subject** (French in this case) returns far better results than English-only searches
- OSINT isn't always about technology — sometimes the most powerful skill is knowing which historical archive to check

---

*Room link: https://tryhackme.com/room/letter*
