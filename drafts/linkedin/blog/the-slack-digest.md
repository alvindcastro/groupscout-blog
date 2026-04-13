I almost built a web dashboard for Group Scout. It would have taken weeks to build and might have been ignored.

My wife, a hotel sales director, already lives in Slack.

Instead of a dashboard, the Group Scout 'UI' is a structured Slack message. Every Monday morning, she receives a Slack digest with:
1. Lead cards built with Slack's Block Kit.
2. Direct links to the original permit PDFs for verification.
3. Emoji tiers (🔥 to 🧊) mapped to Claude’s priority scores.

To keep development fast, I built a `-testslack` flag into my Go smoketest. It sends dummy leads to verify the layout in seconds.

The lesson: Meet your users where they work. A sales manager makes calls; they do not want another dashboard to check.

The system handles the PDFs and APIs. The human handles the sales.

Full breakdown: [Link]
