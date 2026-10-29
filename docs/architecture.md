# Architecture (sanitized)

> This document describes the system at the module level. Prompt engineering
> details, shot-language tables, and provider-specific tuning are intentionally
> omitted — they are the trade secrets that make the product work.

## Stack in one line

`Taro + NutUI + Zustand` (client) ‖ `NestJS + Prisma + BullMQ + LangChain + Logto + Puppeteer` (server)

## Layers

```
Client (H5 / mini-program, one codebase)
  diary editor · illustrated timeline · album export · subscription
        │ HTTPS REST
Server API (shared by all clients)
  users/orders · diary CRUD · generation task queue · album rendering · messaging
        │
Agent orchestration (server-side)
  (1) semantic segmentation → (2) storyboard → (3) prompt refinement → (4) task scheduling
        │
Generation (external APIs)
  LLM API · image API behind a thin adapter (provider switchable)
        │
Data
  users/auth · diary store · object storage (photos + images, pre-signed direct upload) · orders
```

## Module map

| # | Module | Responsibility | Built on |
|---|---|---|---|
| B1 | API framework | REST, middleware, DI | NestJS |
| B2 | Auth | OTP / multi-client identity | Logto (self-hosted) |
| B3 | ORM | users / diaries / orders | Prisma |
| B4 | Task queue | async image generation, retry/timeout/rate-limit | BullMQ + Redis |
| B5 | Segmentation + storyboard | diary text → scene JSON | LangChain + JSON Schema (prompts proprietary) |
| B6 | Image adapter | thin layer over image API, provider-switchable | self-built (<200 lines) |
| B7 | Object storage | photos + generated images, pre-signed upload | MinIO / COS |
| B8 | Payments | orders, subscription state machine | wechat-pay SDK + own state table |
| B9 | Album rendering | HTML → share cards / PDF | Puppeteer |
| B10 | Content safety | text + image moderation | cloud moderation APIs |
| B11 | Messaging | email / template messages | Nodemailer |
| B12 | Admin | users / orders / moderation console | Refine |
| B13 | Observability | errors, analytics, per-task cost log | Sentry + Umami + own cost table |

## The three parts that are actually ours

1. **Event-level segmentation & storyboard** — a 100-1000 word entry becomes 3-8
   scenes via a single LLM call with few-shot examples and strict JSON Schema.
   No fine-tuning; the craft is in the prompt.
2. **Character anchor** — user photos → a neutral front-facing character sheet,
   stored and reused so the protagonist is identical across days and scenes.
3. **Failure detector & silent retry** — every scene is validated after
   generation (consistency, content, artifacts). Failures are re-prompted and
   regenerated inside the queue. Users never see a broken image.

## Cost tracking

Every generation task logs tokens used, image count, and provider. This feeds
the pricing model: entries per month × quality tiers, so heavy users cannot
eat the margin.
