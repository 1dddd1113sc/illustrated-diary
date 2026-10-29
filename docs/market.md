# Market analysis (sanitized)

> Research as of 2026-10. Conclusion: the technology is solved but the product
> category is unclaimed. This is a real market gap with a 12-18 month window.

## Who has tried

| Category | Status | Examples |
|---|---|---|
| Open-source "diary → illustration" | 2-3 toy demos, <10 stars, abandoned | ai-art-picture-diary |
| Open-source "story → consistent multi-image" | mature tech, none aimed at diaries | StoryDiffusion (6.5k stars), ai-comic-factory (1.3k) |
| Commercial diary apps | text AI analysis only, zero event-level illustration | Day One, Rosebud ($12.99/mo), Reflectr |
| Commercial AI diary illustration | exactly one, single image per entry, no character continuity | Mindsera ($14.99/mo) |

## The gap = the opportunity

1. **Event-level segmentation**: nobody splits one entry into 4-8 scenes with
   one illustration each.
2. **Cross-day character consistency**: the protagonist looks the same on day 1
   and day 100 — nobody does this.
3. **Image-first, text-second**: existing products treat images as decoration.
   We invert it into a visual life album.
4. **Payment willingness is proven**: Rosebud/Mindsera validate the diary
   subscription model (~$13-15/month).

## Risks we track openly

- **Retention**: diaries fight human nature; whether illustration improves
  retention is unproven — the MVP exists to measure exactly this.
- **Compliance**: face photos are sensitive personal data; generative AI
  requires filing in some jurisdictions; image APIs reject some real-person edits.
- **Children**: photorealistic images of minors are a hard line for image
  providers — family-diary use cases must route to cartoon avatars instead,
  which affects audience positioning.
- **Cost**: per-image API pricing means heavy users eat margin — pricing must
  cap entries and split quality tiers.

## Why the window is 12-18 months

The generation capability (character-consistent edits via image APIs) became
practical recently. Big diary players have distribution but not this pipeline;
image-native startups have the pipeline but not the diary habit loop. Whoever
proves "illustration improves diary retention" first owns the category.
