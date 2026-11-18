# Setup guide

Time needed: about 45 minutes the first time. Cost: zero.

## 1. Run n8n with Docker

Install Docker Desktop and make sure it is running. Then, in PowerShell or Terminal, run these one at a time:

```
docker volume create n8n_data
```
```
docker run -d --name n8n --restart unless-stopped -p 5678:5678 -e GENERIC_TIMEZONE=Asia/Kolkata -e TZ=Asia/Kolkata -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

Change the timezone to yours. The weekly schedule runs in this timezone. Open http://localhost:5678 and create the owner account.

In the left sidebar, open **Overview** and confirm a **Data tables** tab exists. If not, update the image:

```
docker stop n8n
docker rm n8n
docker pull docker.n8n.io/n8nio/n8n
```

Then run the `docker run` command again. Your data stays in the volume.

## 2. Get a free Gemini key

1. Create an API key in Google AI Studio.
2. Check your project's rate limits there. Free limits are set per project, so confirm your requests-per-day number.
3. List the models your key can use, and pick a Flash-Lite model:

```
curl "https://generativelanguage.googleapis.com/v1beta/models?key=YOUR_KEY"
```

On Windows PowerShell use `curl.exe`. The model ID is the name without the `models/` prefix. Put it in `llm_model` in Settings later.

## 3. Create two credentials in n8n

- **Header Auth**, named `LLM API Key`. Header name: `Authorization`. Value: `Bearer YOUR_KEY`.
- **SMTP**, named `Gmail SMTP`. Host `smtp.gmail.com`, port `465`, SSL on, your Gmail address as user. The password must be a Google **App Password** (2-Step Verification must be on). If port 465 fails, try port 587 with SSL off.

Never paste a key or App Password into a node field, and never commit them.

## 4. Import the workflows, in this order

1. **one-time setup** (`workflows/AI_Release_Tracker_-_one-time_setup.json`). Import it and click Execute workflow once. Check the Data tables tab for `sources` (5 rows) and `llm_log` (empty). Do not run it twice, or the feeds are added twice.
2. **error alert**. Import it, open **Send Failure Email**, choose the `Gmail SMTP` credential and set your addresses. Save, then **Publish** it.
3. **tracker**. Import it, then:
   - Choose `LLM API Key` on **Analyze Item** and **Write Brief**.
   - Choose `Gmail SMTP` on **Send Email**.
   - In **Settings**, set `email_from`, `email_to` and `llm_model`.
   - In Workflow settings, set **Error workflow** to the error alert workflow. It only appears in the list after it is published.

## 5. Test, in this order

1. In **Settings**, check that `test_source` is `samples` (the default). Hover the **Manual Test Run** node and click Execute workflow. No feeds are fetched and it uses about 11 requests. Expect an email with `[TEST]` in the subject.
   Check that the duplicate sample was removed, that each item's tags are close to the `expected` label next to it, and that the rule checks section is mostly PASS.
2. Set `test_source` to `feeds` and run again. This reads the real feeds over 30 days and uses up to `max_items + 1` requests. Read the **COVERAGE** section to see what was left out.
3. Publish the tracker workflow. It then runs on schedule.

## Everyday changes

- **Add a product:** add a row to the `sources` table (company, name, RSS url, enabled).
- **Pause a source:** set `enabled` to false.
- **Switch model or provider:** change `llm_base_url` and `llm_model` in Settings, and update the key in the `LLM API Key` credential.
- **Check token use:** open the `llm_log` table and filter `mode = live`.
- **Change the voice or tags:** edit the prompts in Settings. If you change allowed tag values, also update the lists in the **Rule Checks** node.

## Troubleshooting

| Problem | Fix |
|---|---|
| Error alert dropdown shows workflows greyed out | Only published workflows can be chosen. Publish the error alert workflow first. |
| No error email after a failed manual run | Error workflows only fire for automatic runs. |
| SMTP connection fails | Use an App Password, not your normal password. Match port 465 with SSL on, or port 587 with SSL off. |
| 429 error from the LLM | You hit a free-tier limit. Lower `max_items` or wait a day. |
| A rule check fails | Open the email's RULE CHECKS section. It names the failing check. |
