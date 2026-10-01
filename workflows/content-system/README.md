# Content Engine — n8n Workflows

n8n workflows that turn call transcripts into on-brand content. A transcript goes in, stories are extracted, and each story is written up as a LinkedIn post, blog, video guide and image carousel, all stored in Supabase.

```
Transcript ──► A: Story Extraction ──► stories ──► B: Content Generation ──► content_pieces
                                                                                   │
                                     brands.gold_examples ◄── C: Flag Gold ◄───────┘
```

## Workflows

| File | Trigger | What it does |
|---|---|---|
| `workflow-a-story-extraction.json` | `POST /webhook/extract-stories` | Pulls distinct stories out of a transcript with Claude, dedupes them, saves them to `stories` and marks the transcript as extracted. Retries once on bad JSON and returns 400 or 422 errors to the caller. |
| `workflow-b-content-generation.json` | `POST /webhook/generate-content` | Writes each requested format for a story, renders carousel slides with Gemini and uploads them to Supabase Storage, runs a banned-patterns check and an LLM quality score (under 6/10 is flagged `low_quality`), saves the results to `content_pieces` and calls the app back when it's done. Skips formats that already exist. |
| `workflow-c-flag-gold.json` | `POST /webhook/flag-gold` | Adds a content piece to its brand's `gold_examples`, which are fed into future generations as style references. |
| `ops-health-monitor.json` | Every 15 minutes | Checks n8n's `/healthz/readiness` endpoint and sends an alert email if it isn't healthy. |

### Request bodies

```jsonc
// A — extract-stories
{ "request_id": "…", "transcript_id": "…", "brand_id": "…", "raw_text": "full transcript" }

// B — generate-content  (formats is optional; defaults to all four)
{ "request_id": "…", "story_id": "…", "brand_id": "…",
  "formats": ["linkedin_post", "blog", "video_guide", "carousel"] }

// C — flag-gold
{ "content_piece_id": "…" }
```

## Requirements

- **n8n** (self-hosted or cloud)
- **Supabase**: the tables `brands`, `transcripts`, `stories` and `content_pieces`, plus a public Storage bucket named `carousels`
- **OpenRouter** API key. All LLM calls use `anthropic/claude-sonnet-4.6` through the native AI Agent and OpenRouter chat model nodes.
- **Google AI Studio** API key with billing enabled, for Gemini image generation (`gemini-2.5-flash-image`)
- **SMTP** account, only needed for the health monitor's alerts

## Setup

1. **Import:** in n8n, go to *Workflows → Import from File* and import each JSON file.
2. **Credentials:** every credential ID has been replaced with `REPLACE_ME`. Create these credentials and pick them on each node that shows a warning:

   | Credential name | n8n type | Used in |
   |---|---|---|
   | Supabase account | Supabase API | A, B, C |
   | OpenRouter (native) | OpenRouter API | A, B |
   | Google AI Studio account | Header Auth (`x-goog-api-key`) | B |
   | Webhook Auth | Header Auth (`x-webhook-secret`) | B (app callback) |

3. **Placeholders:** search the imported workflows for these values and replace them with your own:

   | Placeholder | Replace with |
   |---|---|
   | `https://YOUR_PROJECT.supabase.co` | Your Supabase project URL (in workflow B's carousel upload) |
   | `https://YOUR_APP_HOST` | The app that receives the generation-complete callback (in workflow B, the **Notify Replit** node) |
   | `https://YOUR_N8N_HOST` | Your n8n base URL (in the health monitor) |
   | `you@example.com` | Alert sender and recipient addresses (in the health monitor) |

4. **Activate** the workflows. If you're upgrading an existing install and a webhook returns 404 after you redeploy, turn the workflow off and on again to re-register the webhook.

## Securing the webhooks

The webhooks are exported **without authentication**. Before you expose them publicly, set each Webhook node's *Authentication* to **Header Auth**, using a credential that checks an `x-webhook-secret` header, and have the calling app send that header.

## Updating these files

The JSON is exported from a live instance by `export_workflows.py`, which also sanitizes it. The script removes pinned test data, instance IDs, credential IDs, hostnames and email addresses, and it refuses to write any file that still contains a known secret. Don't commit raw exports straight from the n8n UI.
