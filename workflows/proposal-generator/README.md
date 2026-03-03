🤖 Proposal Generator — n8n Workflow
An end-to-end automation that turns a meeting transcript into a polished, client-ready proposal — complete with a Gamma presentation deck, a Gmail draft, and a CRM update — all triggered by a single file upload to Google Drive.
Built by Asero.

✨ What It Does

Detects a new transcript uploaded to a Google Drive folder
Extracts client info (company name, speakers) from the filename and transcript text
Asks you via Slack whether you want a proposal generated — you approve with one click
Queries your Notion CRM to find the matching client record
Handles missing data — if the CRM is incomplete, sends you a Slack alert with a form to fill in the gaps
Generates a full proposal using an AI agent (powered by OpenRouter / Gemini)
Creates a Gamma presentation deck automatically from the proposal text
Notifies you on Slack when the deck is ready with a direct link
Creates a Gmail draft pre-written and addressed to the client
Updates your Notion CRM with the proposal URL, date, and status


🗺️ Workflow Overview
Google Drive (new file)
    └─► Get Transcript
        └─► Extract Client Name
            └─► Slack: "Generate proposal?" ──[No]──► Stop
                └─► [Yes]
                    └─► Query Notion CRM
                        ├─► [Complete] ──────────────────────────────────────────┐
                        └─► [Incomplete]                                          │
                            └─► Slack Alert                                       │
                                └─► Wait for Form Submission                      │
                                    └─► Update CRM                                │
                                        └─► Re-Query CRM ──────────────────────►─┤
                                                                                  │
                                                                    Proposal Generator Agent (AI)
                                                                          └─► Gamma Deck (API)
                                                                              └─► Poll until complete
                                                                                  └─► Slack Notification
                                                                                      └─► Gmail Draft
                                                                                          └─► Update CRM

🧰 Requirements
ServicePurposeGoogle DriveTranscript upload trigger + file exportNotionCRM — stores client recordsSlackApproval prompts + notificationsOpenRouterAI model for proposal generationGammaPresentation deck creationGmailEmail draft creation

⚙️ Setup Guide
Step 1 — Connect Google Drive
Link your Google Drive account in n8n and open the Google Drive Trigger node. Set it to watch the folder where you upload your meeting transcripts. Replace YOUR_GOOGLE_DRIVE_FOLDER_ID with the ID from your folder URL.
Step 2 — Connect Notion CRM
Connect your Notion integration in n8n and replace YOUR_NOTION_DATABASE_ID in both the Query CRM and Re-Query CRM nodes with your database ID (found in the Notion page URL).
Your Notion database needs these properties:
PropertyTypeBusiness NameTitleContact PersonRich TextEmailEmailStatusStatusProposal DateDateProposal URLURL
Step 3 — Connect Slack
Link your Slack workspace in n8n and replace YOUR_SLACK_CHANNEL_ID in all three Slack nodes with the channel ID where you want alerts and approvals sent.
Step 4 — Add Your OpenRouter API Key
Create an account at openrouter.ai, generate an API key, and add it as a credential in n8n. Connect it to the OpenRouter node.
Step 5 — Add Your Gamma API Key
Generate an API key from your Gamma dashboard and paste it into the X-API-KEY header value in both the Gamma Deck and Get Presentation HTTP Request nodes.
Step 6 — Connect Gmail
Link your Gmail account in n8n. The workflow will create a draft — it won't send anything automatically.
Step 7 — Personalise the AI Prompt
Open the Proposal Generator Agent node and replace every instance of YOUR_COMPANY_NAME with your actual company name. Update section 9 ("Why Choose Us") with your own value proposition.
Step 8 — Set Your Name Filter
In the Extract Client Name Code node, replace YOUR_NAME_HERE with your own name (e.g. john smith) so the workflow correctly excludes you from the detected client speaker list.
Step 9 — Set Your Email Sign-Off
In the Create Email Draft node, replace YOUR_NAME in the email body with your own name.

📂 Transcript Naming Convention
The workflow extracts the client company name from the transcript filename. For best results, name your files using one of these formats:
CompanyName x Meeting Notes.docx
Meeting Notes - CompanyName.docx

🔁 How the CRM Fallback Works
If the matched Notion record is missing an email or company name, the workflow:

Sends a Slack message with a link to a fill-in form
Waits up to 7 days for a response
Updates the Notion record with the submitted data
Re-queries the CRM and continues automatically


📝 Notes

The workflow is set to inactive by default — activate it in n8n when you're ready
The Gamma deck uses the vortex theme with 10 slides — customise these in the Gamma Deck node
The AI model is set to google/gemini-3-pro-preview via OpenRouter — swap this for any model available on your OpenRouter plan
No proposal is ever sent automatically — you always review the Gmail draft first


📄 License
MIT — free to use, adapt, and share.
