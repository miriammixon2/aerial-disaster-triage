# Aerial Disaster Triage

Sorts aerial and drone photos from disaster areas into three urgency tiers, so
responders get an order to work in instead of a folder of images.

| Tier | Meaning | Action |
|---|---|---|
| **IMMEDIATE** | Active fire or flooding; someone may be in danger now | Dispatch |
| **DESTRUCTION** | Real damage, but the event is over | Assess when crews are free |
| **NORMAL** | Nothing to respond to | Skip it |

Built for a class on building software with AI assistance. I had no programming
experience when I started.

---

## What's here

- `DEMO_aerial_triage.ipynb` — the working version. Downloads real disaster
  photos, classifies them, prints a ranked dispatch list, and reports its own
  error rate.

Open it in [Google Colab](https://colab.research.google.com), run the cells top
to bottom. No GPU needed. Nothing is installed on your computer.

---

## How it works

There is no trained model. The classifier is a list of plain-English
descriptions and the tier each one belongs to:

```python
"an aerial photo of a building with colored flames or active smoke": "IMMEDIATE"
"an aerial photo of a collapsed building":                           "DESTRUCTION"
"an aerial photo of a lake":                                         "NORMAL"
```

CLIP scores how well each description matches the photo. Scores are summed
within each tier, and the highest tier wins — unless IMMEDIATE clears a
confidence threshold, in which case it escalates.

Twenty-four sentences do all the work. Changing the program means editing
English.

---

## Results

**81% agreement** with the dataset's own labels on 32 real aerial photographs
(26/32), with no training and no tuning on that set.

| Category | Correct | Usual mistake |
|---|---|---|
| Earthquake | 7 / 8 | called it normal |
| Fire | 6 / 8 | called it destruction |
| Flood | 6 / 8 | called it normal |
| Normal | 7 / 8 | false alarm |

Six errors, three different costs: 3 missed a real emergency, 2 identified
damage but misjudged urgency, 1 was a false alarm. A single accuracy number
hides that distinction, and for a triage tool the distinction is the point.

---

## The first version, and why it failed

The project started as change detection: compare a before photo with an after
photo, subtract them pixel by pixel, and flag buildings whose brightness
changed most.

It scored an F1 of 0.777 across 1,350 buildings — which looks reasonable until
you look at what it was measuring.

| True damage level | Average change score (0–255) |
|---|---|
| No damage | 52.7 |
| Minor damage | 72.7 |
| Major damage | 93.0 |
| Destroyed | 145.4 |

The decision cutoff landed at **45**. The average *undamaged* building scored
**52.7** — above the line. It produced 307 false alarms against 726 intact
buildings.

The measurement was honest. It just wasn't measuring damage:

> A shadow moving between two photos changes more pixels than a collapsed roof.

Pixel difference has many causes — the sun moves, trees shift, cars drive off,
seasons change. Subtraction can't tell those from a building falling down. No
threshold fixes that, because the problem isn't where the line sits.

---

## The limitation that survived the rewrite

The current version still can't tell a **burning** house from a **burned-out**
one. Both are char, smoke and a dark roof from the air. One needs a crew right
now; the other doesn't.

I found this failure on generated test images first, predicted it would persist
on real data, and confirmed it — it caused two of the six errors.

Both versions fail the same way underneath: they measure what a scene *looks
like*, not what is *happening* in it.

---

## Limitations

- Three tiers is coarse. Real triage has more gradations.
- Judged against a research dataset's labels, not against what a dispatcher
  would actually decide.
- Sample size is small; one photo moves the score by about 3%.
- No geolocation. Filenames stand in for addresses.
- Not suitable for real emergency use.

---

## Data

[AIDERv2](https://huggingface.co/datasets/ckyrkou/AIDERv2) — 16,723 real aerial
and drone images across Fire, Flood, Earthquake and Normal, collected from news
footage, online sources and the researchers' own UAV flights.

> Shianios, D., Kyrkou, C., and Kolios, P. S. *A Benchmark and Investigation of
> Deep-Learning-Based Techniques for Detecting Natural Disasters in Aerial
> Images.* Computer Analysis of Images and Patterns (CAIP), 2023.

Released for non-commercial research use. The notebook downloads a sample at
runtime; no images are stored in this repository.

Model: [CLIP ViT-B/32](https://huggingface.co/openai/clip-vit-base-patch32) (OpenAI).
