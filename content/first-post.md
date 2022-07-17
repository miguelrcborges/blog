---
title: "First Post"
date: 2022-07-17T01:03:20+01:00
draft: false
---

# I deleted my past posts

I did a system reinstall and noticed that pandoc is bloated, if you don't use haskell and it's dependencies (around 500 MBs).

I ended up replacing pandoc with Hugo, because it's:
* Written in Go, one of the languages that I like the most;
* Extensible;
* Relatively lightwheight (50 MBs total); 

## My experience on remaking the blog on Hugo

It was confusing at first. I wasn't being able to find how I would theme it without relying on a theme.
Since I only was aware that styling was only available through themes, I searched how to make my own,
where I found out the folder structure of making a theme coincides with the site's. 

The next weird thing was finding out how pages rendered.
If you have a `index.html` in `layouts` it will override the generated one.
Hugo, to generate the files, relies on 3 .html files:
* `layouts/_default/baseof.html` - The base html for everypage.
* `layouts/_default/list.html` - Processed on the index page.
* `layouts/_default/single.html` - Processed on every file inside the `content` directory.

I had mention "\_default" on every directory, but if you have markdown files inside different directories of `content`, you can specify different rules
by adding a folder in `layouts` with the same folder name, instead of "\_default".

After figuring how it works, the experience has been nice.


## What about the other posts?

To be honest, they sucked, so I didn't bother porting them. They are still accessible if you check the past commits, but I don't recommend the hassle.

Once I'm now on holidays, I've more free time to explore whatever I wish and I can spend more time on improving the text quality of the blog.

Thank you for the read and stay tuned for the next content :)
