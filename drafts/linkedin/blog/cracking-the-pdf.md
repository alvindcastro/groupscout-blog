PDFs are where data goes to die.

I am building a system to track municipal building permits. Most cities provide weekly reports as PDFs instead of APIs.

Three lessons from cracking them:
1. Do not fight encoding. I used `pdftotext`, a tool that solved this problem twenty years ago.
2. Trust content over position. My first parser relied on line numbers. A single permit count shifted every field. I switched to regex—if it looks like a date, it is a date.
3. Test with real data. My unit tests passed because I used clean data. The real output was messy.

Municipal data is inconvenient, but it is a goldmine if you can extract it.

The full story: [Link]
