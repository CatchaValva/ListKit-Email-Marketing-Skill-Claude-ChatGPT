LeadnClose AI
A self-hosted business outreach worker for Windows, macOS, or Linux. It finds published business email addresses, sends your campaign copy from your mailbox, and forwards interested replies to an inbox you choose. It uses Python and SQLite. ChatGPT/Codex or Claude Code can work with the repository, but the worker runs independently and uses no paid AI API.
Install on Windows
Open PowerShell in the extracted project folder. Install Python 3.10 or later first if you do not already have it.
```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
Copy-Item config.example.json config.json
notepad config.json
```
Use `python3` to create the environment on macOS/Linux, then use `.venv/bin/python` for the commands below.
Discover contacts
In `config.json`, set `searches` to one or more location bounds and business tags. The example searches for physiotherapy businesses around Houston. The bounding box order is south, west, north, east. Choose tags from the OpenStreetMap tagging wiki. You can also put known public HTTPS business websites in `websites`. The optional `keyword` filters mapped business names within the directory results. Written addresses are collected from public map tags, the business homepage, and up to two same-host contact pages.
```powershell
.\.venv\Scripts\python.exe leadnclose.py discover
.\.venv\Scripts\python.exe leadnclose.py leads
```
Records show the business, address, source, approval status and mail-domain status. Discovery focuses on published role addresses such as info@ and sales@. A domain with MX records is eligible for campaign enrollment. Individual inbox existence, catch-all behavior and delivery remain unconfirmed. Coverage varies by geography and business type, and some websites require JavaScript or block automated requests.
Lead discovery uses the public Overpass API. Map data © OpenStreetMap contributors. Keep queries narrow and follow the service’s usage policy.
Connect your sending mailbox
Use a dedicated campaign mailbox. Set `from`, `username`, SMTP/IMAP hosts and ports, `sender_identity`, your business `postal_address` and the default `forward_to` inbox in the local config. Each campaign can override the forwarding inbox with its own `forward_to`. The receiving inbox can be hosted by a different provider.
App password or password-based provider. Use the provider’s TLS SMTP and IMAP settings. Supply the credential through the local environment variable `LEADNCLOSE_MAIL_SECRET`. The script uses implicit TLS on SMTP port 465 and STARTTLS on other SMTP ports. On PowerShell 7, enter your credential locally with:
```powershell
$env:LEADNCLOSE_MAIL_SECRET = Read-Host "Mail app password" -MaskInput
```
Outlook.com or Microsoft 365. Add `microsoft_client_id` from your Microsoft Entra public-client application registration and, when needed, set `microsoft_tenant` to your tenant ID. Use an app registration that supports your account type and permits public-client device-code flow. Enable delegated Exchange Online permissions `IMAP.AccessAsUser.All` and `SMTP.Send`, and make sure the sending mailbox allows these protocols. Personal Outlook.com users can use IMAP host `outlook.office365.com`, and SMTP host `smtp-mail.outlook.com` on port 587. Microsoft 365 users should use their organization’s settings. Complete Microsoft sign-in locally:
```powershell
.\.venv\Scripts\python.exe leadnclose.py microsoft-login
```
Microsoft publishes Outlook.com server settings, OAuth protocol setup and Exchange Online SMTP AUTH settings. Some organizations require administrator consent or block device-code flow. Keep the local token cache restricted to your operating-system account.
Start a campaign
Approve the business contact IDs you are eligible to email. The example below uses IDs 3 and 7.
Write the subject and body in `config.json`. Templates support `{{company}}` and `{{email}}`. Set `lead_ids` to the approved IDs for the campaign. You can omit `lead_ids` to include all approved leads matching the optional `company_contains` filter, including future approvals. Give each campaign a unique ID. Enable it with `"enabled": true`. An optional `start_at` accepts a timezone-aware ISO 8601 timestamp.
Preview the eligible recipients, then launch the worker when ready.
```powershell
.\.venv\Scripts\python.exe leadnclose.py approve 3 7
.\.venv\Scripts\python.exe leadnclose.py send
.\.venv\Scripts\python.exe leadnclose.py run --live
```
Start with one address you control. A reply such as “Yes!” or “Please send more details” should reach the configured forwarding inbox after the next poll. Forwarded emails contain the reply text and use the prospect as Reply-To. The worker records uncertain replies as `review`. It records opt-outs and suppresses those addresses across all campaigns. You can suppress a lead manually with `suppress ID`.
`run --live` repeats discovery, checks replies and sends approved campaign emails while one worker process is running. Restart it after editing the config. A computer that sleeps or shuts down pauses the worker. The default cap is 25 outreach attempts per UTC day. Use a limit permitted by your mail provider.
```powershell
.\.venv\Scripts\python.exe leadnclose.py deliveries
.\.venv\Scripts\python.exe leadnclose.py replies
```
Delivery records report `sent` after SMTP accepts the submission. If an SMTP operation fails after being reserved, it remains `attempted` for inspection so a timeout after acceptance cannot automatically cause a duplicate. Reply matching requires the campaign’s message ID in the reply headers and the same sender address. Responses sent from a different address, new threads and attachments sit outside this workflow. The rule-based classifier handles common English signals and requires human review for uncertain language. Keep credentials, SQLite data, and real replies local.
Verification and source control
Run `python -m unittest -v` from the project directory. The tests cover lead discovery and deduplication, approval, preview, campaign delivery, reply threading, classification, forwarding and opt-outs using fixtures and mocked SMTP. They require no credentials. Live external integrations require your mailbox and network.
