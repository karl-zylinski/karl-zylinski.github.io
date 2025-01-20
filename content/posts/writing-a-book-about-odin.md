---
title: "Writing a book about Odin"
date: 2025-01-20T11:40:40+01:00
draft: true

#cover:
#  image: "/game_go_brr.png"
---

In this post I'll share some details on how I wrote the first book about the Odin Programming Language. I'll tell some "how I got started" stories as well as some technical things related to writing programming books. 

> This book is called "Understanding the Odin Programming" language. It's a digital book (HTML and ePub). It was released on December 6, 2024. The book is available here: https://odinbook.com/

## A happy accident

Exactly how one gets started on a project varies. I didn't plan to write a book until I was already in the middle of doing it. It started with me writing an introductory blog post about Odin. I thought that it would be nice to write something that is reader-friendly and also tries to highlight interesting things about the language.

The result of that is the blog post "Introduction to the Odin Programming Language". It took a few days to write. You can read it here: https://zylinski.se/posts/introduction-to-odin/

My creative projects often seem to start with some kind of "happy accident": Something small that leads to something bigger. I didn't know I had written the seed a whole book, and that I would be working on it for many months.

## Book ideas

Quite quickly after posting the blog post, I was already thinking about writing something bigger. Writing the blog post had come quite naturally to me, and I found it fun. I also [posted the blog post on Twitter](https://x.com/karl_zylinski/status/1799829568590950779), where people quickly noted that I should turn it into a book.

However, I wasn't sure what a book would mean. A physical book? An e-book? What's the target audience? What does it try to teach? Do I want to write a book?

I quite quickly figured out the answer to "What does it try to teach?". I've helped many beginners on the Odin Discord server (the official Odin chat) by answering their questions. I've noticed some questions that were hard to answer without explaining several separate ideas that were all key in intuitively understanding what they were asking. Some of these things were not covered in the official documentation, and if they were, there was often no "connection between the dots": You needed to spend a lot of time figuring things out by yourself and see how things were connected. That's what I wanted to fix: Bridge the gap between the online documentation and years of using Odin, hopefully saving the reader a lot of time.

When I had that figured out, I just started writing. The target audience I would refine over time, I wanted to try writing a few different chapters and see what style I wanted to go for. I hadn't yet decided on if I wanted a physical release, but I figured a digital release (some kind of e-book) would be good starting point.

## Choosing technical tools

How does one write a digital book? What software do you use? How does the text become the e-book, or a physical book for that matter?

Early on, I realized that I wanted this digital book to be available in HTML format. People who are learning a programming language are likely to be reading on their computer screen, next to their code. I didn't want to release a PDF, because I really dislike reading PDFs on a computer screen. A HTML page is usually much nicer!

So here's what I did: I took the intro-blog-post I had written, and I used that as a starting point. But I wanted my book too work in a very custom way. I wanted:
- Complete control over how text was laid out
- Small "bubbles" on the side with extra info
- No javascript
- As few files as possible, preferrably a single HTML file.

For the last point, I didn't know if it was possible when I started. But I knew if I wanted to get there, then I needed a lot of control over how the HTML was assembled.

My blog uses hugo, which is a website generator. The blog post is written in markdown. I wanted to write my whole book in markdown. But in order to get all this control, I felt like I should ditch hugo for generating the book. This was the best decision I did in this whole project.

## Markdown!

I started writing an Odin program that took my markdown, parsed it and constructed a HTML file from it. I had a `header.html` and `footer.html` file and then the book contents goes in-between.

I just started by splitting the markdown into lines and checking what each line started with:

- `# Heading` -> this line starts with `#` so the text on the line is enclosed in a `<h1>` tag.
- `## Heading` -> `<h2>` tag
- With separated by line breaks: Group as paragraphs, i.e. enclose with `<p>`

There is something to note here: Since I only needed to support a subset of markdown that I actually used, then I could cut _a lot_ of corners. I didn't even write proper tokenization. I just check what a line starts with, if there are line breaks, etc.

After a while I needed to support `code`, *strong* and _italic_. So then I wrote a small procedure that I sent the whole paragraph into that scanned for the symbols that represent those kind of things: \`, `_` and `*`.

Quite soon, I had a program that could output my original blog post, but with full control over how it did it. I then added the table of contents, that went on the side of the screen. That was done by just saving away some metadata each time I parsed a line starting with one or more `#`.

## Side-bubbles

One exciting thing I did quite soon thereafter was to add the "side bubbles": Extra information that live in a little bubble on the side. I did some CSS stuff to make those bubbles have a little arrow. Since the view is of fixed width, I could even adjust their position to point at the perfect place in the text. In normal markdown the following is a block quote:

```
> This is a block quote.
```

But I made it so that all those `>` ended up in these side-bubbles.

I could fine-tune the bubble's position by writing a number directly after `>`:

```
>30 This bubble will be pushed up 30 pixels on the screen.
```

## Responsive CSS

If you make your screen too narrow, then there is no room for the side-bubbles. When it gets to that point I switch to a view where the side-bubbles end up beneath the text.

> That's more like these classic blog block quotes.

## Inspiration from Bob Nystrom

I reached out to Bob Nystrom, author or "Crafting Interpreters". I asked him about how to publish books. He said that he did almost everything himself and linked to his blog, where there are some posts about how he made his books: https://journal.stuffwithstuff.com/category/book/

His style really resonated with the way I was working. He had also made his own script that assembled a website, and everything was custom-made for total control. It was nice to know I wasn't alone in my quest for total control.

## Bundling everything inside the HTML file

If you've bought the HTML version of my book, then you might have noticed that it is a single HTML file. This HTML file is around 3 megabytes! The size is due to all the images and fonts being bundled inside it. It's like a PDF, but nicer to use!

The images are added in the markdown as normal markdown image tags:
```
![Shows how you can swizzle a 4D vector into a 2D vector](swizzle.jpg)
```

When the book-building program runs into this tag, then it loads the image (`swizzle.jpg`) in this case and opens it. It then base64 encodes the image data (using `import "core:encoding/base64"`). The base64 encoded data is put into an image tag:

```
<img src="data:image/FILE_FORMAT;base64,DATA" alt="ALT_TEXT">
```

So for a jpeg it says `image/jpeg` and `DATA` is replaced with the result of the base64 encoded data. That way I was able to embed every single image into the HTML.

It's also very important to bundle the fonts: If someone's internet fails, then the book will look very bad. Also, the side-bubbles are positioned quite carefully, so it's important that the correct font is always used.

To accomplish this I made a `fonts.css` file, which contains the data of the three fonts I use, base64 encoded! You can do that in CSS like this:

```
@font-face {
	font-family: "Font Name";
	src: url('data:font/ttf;charset=utf-8;base64,DATA') format('truetype');
}
```

With that fixed, I had my whole book in a single, portable HTML file!

## Non-bundled samples

The HTML file with everything bundled was only for making it easy to download and store the book when you buy it. Sort-of how you can download a single PDF and get everything.

However, I wanted an online sample that did _not_ have the bundled images and fonts. Bundling them inside an online sample would put a lot of stress on the server, since I doubt it can cache the base64 encoded data. You can see the sample here: https://odinbook.com/sample.html It is generated with a special flag that only includes the first three chapters (it also does some extra hard-coded stuff, like adding the "End of sample. Thanks for reading! blabla" text). For the sample I just generate normal image tags and copy the images to the output directory instead. The fonts are supplied by google fonts. That way I minimize traffic for the sample!

## EPUB

I also wanted to put the book on Amazon and Google Books. The HTML version is The Premium Reading Experience. But I can't put that on an eReader (Amazon Kindle, Kobo etc) or in the Google Books store. So I needed to construct an EPUB version.

[Bob wrote a bit about](https://journal.stuffwithstuff.com/2014/11/03/bringing-my-web-book-to-print-and-ebook/) how annoying EPUB is. I recommend that you read his blog for information about that. Here I give some additional information and tricks.

### Zipping the EPUB

Bob wrote in his blog post that an EPUB is basically a zip. He wrote that you can run the following to make the zip:
```
cd my_awesome_site
zip -X0 my_awesome_site.epub mimetype
zip -Xur9D my_awesome_site.epub *
```


### What goes into the EPUB?

I recommend that you go to gutenberg.org and download yourself some public domain EPUB, unzip it and study it. I used Frankenstein: https://www.gutenberg.org/ebooks/84

> For the longest time, my book had the Frankenstein cover...

### Minimal EPUB styling

The same program that generates the HTML version generates the EPUB version. It uses a completely separate CSS file however. An EPUB should have as little styling as possible, so the user can configure their reading experience through their eReader.


### Don't bundle fonts on EPUB

My book took like 2 minutes to load on my kindle, because I had bundled the fonts. I didn't need the fonts, I had just forgotten to remove the CSS file that contained the base64 encoded fonts. Don't do that. The eReader will decide on fonts itself.



One VERY STRANGE THING about EPUB is that it's _important_ that the mimetype file is added first. Why? Because the EPUB format goes in and looks for it _at the start of the zip_. It needs it to be first as some kind of "header". Absolutely bonkers.



I've been on the [Odin Discord](https://discord.gg/odinlang) (the official Odin chat). In there I often try to help people in the #beginners channel. Over time, I've noted some common questions. Some of those common questions are sometimes complicated to answer. Some answers require you to explain several separate concepts, in order give the person a good intuition. That's hard to do in a chat.

Many of these things I _had_ to explain in the chat, because there were no documentation online that was on a beginner's level that also "connected the dots" between the different concepts involved in answer a complicated questions.

So I thought that I should try to collect my ideas in a blog post! I just started writing out different things that I thought were unique about Odin and also tried to answer some of 

https://zylinski.se/posts/introduction-to-odin/


On December 6, 2024 I released "Understanding the Odin Programming Language". It's the first book on Odin.