I built the first end-to-end pipeline for Group Scout in one weekend.

I focused on the 'happy path' to prove the concept:
1. Fetch a PDF from a static URL.
2. Extract text using `pdftotext`.
3. Regex for the first five projects.
4. Pass the text to Claude for scoring.
5. Push a JSON payload to Slack.

This 'toy' pipeline was fragile. It failed if the PDF was missing or if Claude returned malformed JSON. But it proved that I could extract a lead from a PDF and score it with AI.

The first working pipeline is not about scale; it is about visibility. Once I saw a lead in Slack, I knew the project was worth building.

Engineering breakdown: [Link]
