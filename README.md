# signal-sifter
Sorts real brand mentions from coincidental word-match noise, using Jev. Built for anyone whose product name collides with a common word or another company.

Powered by TypeSafe's Jev, a fast decision-model API.

Try it live: [paste your GitHub Pages link here once deployed]

**The problem**

If your product's name is also a common word — or worse, shared by an entirely different company — your alert feed is mostly noise. This isn't a made-up pain point: Brandwatch, Mentionlytics, and BrandMentions all sell "Boolean query filtering" as a headline feature, built for exactly this. This tool does the same job with a plain-English description instead of a query language, using Jev to judge each mention on meaning rather than keyword match.

**What's in this repo**

• index.html — the page. Paste your Jev API key, what you're tracking, what it actually means, and a batch of mention snippets. It sorts them into "real signal" and "noise."
• signal-sifter-auto.json — the n8n workflow behind it. One workflow, two jobs:
• A webhook the page calls (the part you interact with directly)
• A scheduled branch that reads a real Google Alerts RSS feed and posts real hits straight to Slack, unattended

**Two ways to use this**
1. Just try it

Open index.html (or the live link above), paste your own Jev API key, and run it. Your key is sent per-request and never stored anywhere — not in this page, not on any server. It only leaves your browser to reach the workflow below, which forwards it straight to TypeSafe.

2. Run your own copy end-to-end

Because browsers can't call TypeSafe's API directly (CORS), this needs a small relay in the middle. 

To run your own:

• **Import the workflow.** In n8n: + Add workflow → ⋯ → Import from File → select signal-sifter-auto.json.
• **Activate it (toggle top-right)**, then open the Demo Webhook (from page) node and copy its Production URL.
• Edit index.html: find the line const RELAY_URL = '...' near the top of the <script> tag, and paste your URL in.
• Host it. Push to a GitHub repo, then Settings → Pages → Deploy from branch (main, /root). Give it a minute and your link will be live.

In workflow Settings, turn off Save manual executions and Save execution progress — this workflow passes real API keys through per request, and you don't want those sitting in your execution logs.

That covers the "try it / demo" path — no Slack or Google Alerts needed.

**Optional**: the real unattended pipeline

To make the scheduled branch actually alert you on real incoming mentions:

• Open the Prepare Items node, scroll to the Branch B section, and fill in your own TOPIC, CONTEXT, and Jev API_KEY (this branch runs with nobody typing a key in, so it needs one stored here).
• Set up a Google Alert for your keyword, choose RSS feed as the delivery method, and paste that feed URL into the Read Google Alerts RSS node.
• Open the Post to Slack node, connect your own Slack credential, and set your channel.
• Activate the workflow — it now checks your feed every 6 hours and pings Slack only for the real hits.

**Notes**
• Nothing you paste into the page is stored anywhere but your own browser (localStorage, for convenience only — your last-used key).
• The workflow's "Skip Already-Seen Items" node stops the same RSS entry from re-triggering Slack every 6 hours.
• If any node shows an "update this node" prompt after import, that's normal — n8n versions the node types; just accept the update.
