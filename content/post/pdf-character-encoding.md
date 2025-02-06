---
title: "Pdf Character Encoding"
date: 2024-10-21T16:00:15+02:00
draft: true
---

- What I've been doing
- What I ran in to
- What I did to resolve the issue
- What worked
- Why did it work and what could I have done instead.

# Keeping track of my grocery expenses

I've spent some time downloading my grocery receipts from the API that the store I shop at provides as a backend to their website. It's quite easy to automate due to a couple of technical choices made by the developers, which is nice, I like having access to my data. I did however run into issues trying to parse the PDF-files.

# This is gibberish

So I started to read the files and there are a few nice libraries available. For .NET I like to use [PDFSharp](https://github.com/empira/PDFsharp) which is MIT-licensed and has a nice set of features. I've also used the pdf-crate for rust. But both libraries work well and for the receipts they both give the same result when trying to extract test: an incomprehensible string of characters. Just dumping the text from the files would result in something like:

```
720$7(5&2&7$,/.9,6
*52''$5%1*
&+$03,1-21(5*
%/$'6(//(5,
```

That should start with

```
TOMATER COCTAIL KVIS
```

It turns out that simply copying and pasting the text from a pdf viewer into a text editor produces the same result, it's not some issue with whatever code I've written. If you're perceptive you might also notice that this is more or less a substitution cipher. '7' in this example is clearly 'T' and '2' is 'O' and so on.
Looking a bit closer, it seems that the offset for capital letters is 29. A is encoded as 0x24 (36 in base 10), while it would have been 0x41 (65 in base 10) if it was UTF-8.

If we extract the font from the PDF-file and take a look at it with [FontForge](https://fontforge.org/en-US/), we see how it works. The indexes of the characters in the font correspond to the strange encoding in the PDF:

![FontForge screenshot](/fontforge.png)

With this information in hand, it's easy to do a bit more googling an realize that what's missing from the PDF what's called a ToUnicode-map.
