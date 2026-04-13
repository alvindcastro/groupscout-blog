### Post 2: The "Technical Reality" Post (Focus on the PDF Struggle)
*Best for: Showing the "messy" side of building and why simple solutions (like CLI tools) often win over fancy libraries.*

I thought parsing municipal building permits would be the easy part.

It wasn’t.

Most cities publish their weekly permit reports as PDFs. When I started building Group Scout, I reached for a pure Go PDF library. It returned zero rows. No error, no panic—just silence.

It turns out these PDFs use non-standard font encoding that many modern libraries just can’t touch.

The fix? I stopped fighting the encoding and started shelling out to `pdftotext` (part of the Poppler library). It’s a 20-year-old CLI tool that handles every encoding edge case perfectly.

One line of Go: `exec.Command("pdftotext", "-layout", path, "-").Output()`

Sometimes the best "modern" solution is a battle-tested tool that’s been solving the problem since before Go existed.

More on how I’m "cracking the PDF" here:
https://alvindcastro.github.io/groupscout-blog/posts/cracking-the-pdf/
