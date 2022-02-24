---
title: "Moving to Hugo"
date: "2021-12-27"
draft: false
tags:
  - migration
  - hugo
  - blogger
  - website
---

## Table of Content

- [Table of Content](#table-of-content)
- [Web hosting platform](#web-hosting-platform)
- [Migration preparations](#migration-preparations)
  - [Folder preparations](#folder-preparations)
- [Cleanup and posting on production site](#cleanup-and-posting-on-production-site)
- [Next steps](#next-steps)

Original site was located at [Blogger CloudAlbania](http://cloudalbania.blogspot.com/) but that platform does not provide the editing tools I needed for text processing, e.g. even a simple code formatting took forever.

Here I will post the migration steps in Hugo.

## Web hosting platform

Since Hugo pages are static ones, I really needed some simple webhosting that I could place the `/public` folder. From hugo's [hosting & deployment](https://gohugo.io/categories/hosting-and-deployment) recommendations I found that [Render](https://gohugo.io/hosting-and-deployment/hosting-on-render/) was the best choice so far.

To start migrating I created a git repo in [GitHub](https://github.com/besmirzanaj/cloudalbania-website) first so I could track changes and allow automation of static web pages publishing through Render.

## Migration preparations

After finding an appropriate theme in Hugo ([m10c](https://github.com/vaga/hugo-theme-m10c) by the way) I needed to migrate from [blogger](https://gohugo.io/tools/migrations/#blogger) to hugo MD files. I chose this one as I hate managing html or CSS.

The tool used was [blog2md](https://github.com/palaniraja/blog2md) and the process was fairly easy on my Ubuntu 18.04 on WSL1. The project is based on node.

### Folder preparations

In my hugo project folder I added a new folder ``migration`` and that was my workdir.

```bash
cd your-hugo-project
mkdir migration
sudo apt install npm nodejs git
git clone https://github.com/palaniraja/blog2md.git .
npm install
```

Now we need a backup of the original site in a XML format. In your blogger admin portal, go to _Settings –> Other –> Import & back up –> Back up content_.
Save the XML in the same ``migration`` folder.

For Blogger imports, blog posts and comments (as separate file `<postname>-comments.md`) will be created in "`out`" directory

```bash
node index.js b your-blogger-backup-export.xml out
```

## Cleanup and posting on production site

I had to clean up some old posts so I took to the time to review what was needed and then migrate the newly generated .md files in my ``content/posts`` folder.

Don't forget to either clean up the migration folder or add a new .gitignore entry so that it does not get pushed in your git repo

## Next steps

`blog2md` does not import images from blogger. Images need to be copied over as [static files](https://gohugo.io/content-management/static-files/) in order to fully remove blogger's dependencies.
