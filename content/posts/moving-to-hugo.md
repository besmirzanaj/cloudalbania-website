---
title: "Moving to Hugo"
date: 2021-06-14T10:29:53-04:00
draft: false
---
Oroginal site was located at [Blogger CloudAlbania](http://cloudalbania.blogspot.com/) but that platform does not provide the editing tools I needed for text processing, e.g. even a simple code formatting took forever.

Here I will post the import and editing process in Hugo.

## Migration preparations

After finding an appropriate them in Hugo ([Soho](https://themes.gohugo.io/themes/soho/) by the way) I needed to migrate from [blogger](https://gohugo.io/tools/migrations/#blogger) to hugo MD files.

The tool used was [blog2md](https://github.com/palaniraja/blog2md) and the process was faily easy on my Ubuntu 18.04 on WSL1. The project is based on node.

### Folder preparations

In my hugo project folder I added a new folder ``migration`` and that was my workdir.

```bash
cd your-hugo-project
mkdir migration
sudo apt install npm nodejs git
git clone https://github.com/palaniraja/blog2md.git .
npm install
```

Now we need a backup of the original site in a XML format. In your blogger admin portal, go to Settings –> Other –> Import & back up –> Back up content.
Save the XML in the same ``migration`` folder.

For Blogger imports, blog posts and comments (as seperate file `<postname>-comments.md`) will be created in "`out`" directory

```bash
node index.js b your-blogger-backup-export.xml out
```

## cleanup and posting on production site
I had to clean up some old posts so I took to the time to review what was needed and then migrate the newly generated .md files in my ``content/posts`` folder.

Don't forget to either clean up the migration folder or add a new .gitignore entry so that it does not get pushed in your git repo

## Next steps

blog2md does not import images from blogger. Images need to be copied over in order to fully remove blogger's dependecies.