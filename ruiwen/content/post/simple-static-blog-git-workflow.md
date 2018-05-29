---
Categories:
- Hugo
- Blogging
- Github
- Git
Description: ""
Tags:
- hugo
- blogging
- github
- git
date: 2018-05-29T22:45:00+08:00
title: A Single Repo for Statically Generated Sites on Github Pages
---



There's a lot of talk about how easy it is to setup a statically-generated blog (such as this!) using a static site generator and hosting it on Github pages.

In this post, we'll take a look at a simple `git` workflow that can be used to manage a statically-generated site, such as this.

Github pages usually publish the content from the `master` branch of a git repo, and while some tips found online might suggest keeping the generated content and the source material in separate repos, it's easier to have them all in one project that can be managed all at the same time.

_For the rest of this post, we'll use the [examplepage repo](https://github.com/ruiwen/examplepage.github.io) as an example. and assume we'll be using the [Hugo](https://gohugo.io) blog generator._

# Basic Setup

We'll first setup a new repo on Github to host the site. In this example, we're using `https://github.com/ruiwen/examplepage.github.io`.

We'll also need to ensure that we [have Hugo installed](https://gohugo.io/getting-started/installing/).


With that done, we can begin to setup our project directory. `hugo` provides a quick way to get started with that

    $ hugo new site -f yaml examplepage.github.io
    Congratulations! Your new Hugo site is created in /home/ruiwen/projects/examplepage.github.io.

    Just a few more steps and you're ready to go:

    1. Download a theme into the same-named folder.
       Choose a theme from https://themes.gohugo.io/, or
       create your own with the "hugo new theme <THEMENAME>" command.
    2. Perhaps you want to add some content. You can add single files
       with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
    3. Start the built-in live server via "hugo server".

    Visit https://gohugo.io/ for quickstart guide and full documentation.


    $ ls examplepage.github.io/
    archetypes  config.yaml  content  data  layouts  static  themes


_We use the `-f yaml` option to specify that we'd like to use YAML as a configuration format. `hugo` defaults to TOML by default._

To get started, you'll need to [configure hugo](https://gohugo.io/getting-started/configuration/), setting values for the `baseURL`, and `title` of the site, but that's beyond the scope of this blog post.

For now, let's also create a simple blog post.

    $ hugo new post/a-sample-post.md
    /home/ruiwen/projects/examplepage.github.io/content/post/a-sample-post.md created

You'll notice that new content is rooted at the `content/` directory, and all we need to do is to specify the path under that, eg. ` post/a-sample-post.md`.

We'll also add some content to the new post.

    $ echo "Here's a sample post! Welcome to our sample page!" >> content/post/a-sample-post.md
    $ cat content/posts/a-sample-post.md
    ---
    title: "A Sample Post"
    date: 2018-05-29T17:57:14+08:00
    draft: true
    ---

    Here's a sample post! Welcome to our sample page!


Before we publish the post, we'll need to [configure a theme](https://gohugo.io/getting-started/quick-start/#step-3-add-a-theme) for the site. Check out [themes.gohugo.io](https://themes.gohugo.io/) for themes. In our example, we're using [`hyde-hyde`](https://themes.gohugo.io/hyde-hyde/).

    $ pwd
    /home/ruiwen/projects/examplepage.github.io
    $ git clone https://github.com/htr3n/hyde-hyde.git themes/hyde-hyde
    Cloning into './hyde-hyde'...
    remote: Counting objects: 597, done.
    remote: Compressing objects: 100% (4/4), done.
    remote: Total 597 (delta 0), reused 2 (delta 0), pack-reused 592
    Receiving objects: 100% (597/597), 1.89 MiB | 809.00 KiB/s, done.
    Resolving deltas: 100% (310/310), done.
    Checking connectivity... done.

Next, edit `config.yaml` at the root of the project, and include `theme: hyde-hyde` at the bottom.

At this point, we're almost ready to publish the blog! We just need to remove the `draft: true` line in the post, and we're good to go.

To see what your site would look like, run the dev server, and navigate to `http://localhost:1313`

    $ hugo server


# Committing to `git`

Next up, we'll want to set up our git repo.

    $ git init
    Initialized empty Git repository in /home/ruiwen/projects/examplepage.github.io/.git/
    $ git remote add github https://github.com/ruiwen/examplepage.github.io

First, we want to create an initial empty commit. This will be our repo's starting point, and allow us to manage two different aspects of the site in the same repo.

    $ git commit --allow-empty -m "Initial empty commit"
    [master (root-commit) d9c0248] Initial empty commit

We'll want to create a new branch, that is *not* `master`. For this example, we use `source`

    $ git checkout -b source
    Switched to a new branch 'source'

Then we add all the files

    $ git add .
    $ git commit -m "New post: A sample post"
    [source 2f14616] New post: A sample post
    4 files changed, 21 insertions(+)
    create mode 100644 archetypes/default.md
    create mode 100644 config.yaml
    create mode 100644 content/post/a-sample-post.md
    create mode 160000 themes/hyde-hyde
    $ git log --oneline --graph --decorate --all
    * 2f14616 New post: A sample post  (HEAD -> source) [Ruiwen Chua 1 second ago]
    * d9c0248 Initial empty commit  (master) [Ruiwen Chua 60 seconds ago]

We see that the source files have been committed to the `source` branch.

Now, because Github pages deploys content from the `master` branch, we want a) the fully rendered static files to reside in the `master` branch, and b) none of the source files to reside in the `master` branch. Hugo renders to `public/` by default, so we'll want make sure that our `public/` directory represents our `master` branch.


# Setting `public/` as our `master` branch

We'll first make `public/` the home of our `master` branch. `git` has a wonderful tool that allows us to checkout a branch into its own directory that resides right alongside the rest of the repo. We'll use `git worktree` to make this happen.

    $ git worktree add -b master public
    Preparing public (identifier public)
    HEAD is now at d9c0248 Initial empty commit

The interesting thing about `git worktree` is that it allows us to check out a completely different branch of the repo and have it checked out alongside any other branch we happen to be working on.

    $ cd public
    $ ls
    $ git branch
    * master
      source

In the `public/` directory, we don't see any files in the listing, because the `master` branch is currently pointing at our original empty commit. However, we do see that `git` recognises that we are, in fact, in the `master` branch, and not the `source` branch, where we were previously.

Now that `public/` represents our `master` branch, we're going to get `hugo` to render our site into it.

    $ hugo
                       | EN
    +------------------+----+
      Pages            |  7
      Paginator pages  |  0
      Non-page files   |  0
      Static files     |  8
      Processed images |  0
      Aliases          |  0
      Sitemaps         |  1
      Cleaned          |  0

    Total in 239 ms
    $ ls public/
    404.html  apple-touch-icon-144-precomposed.png  categories  css  favicon.png  img  index.html  index.xml  sitemap.xml  tags
    $ cd public
    $ git status
    On branch master
    Untracked files:
      (use "git add <file>..." to include in what will be committed)

            404.html
            apple-touch-icon-144-precomposed.png
            categories/
            css/
            favicon.png
            img/
            index.html
            index.xml
            sitemap.xml
            tags/

    nothing added to commit but untracked files present (use "git add" to track)

We'll just commit the files in `public/`, and they should be added to our `master` branch

    $ git add .
    $ git ci -m "Publish: 20180529"
    [master ff17216] Publish: 20180529
     16 files changed, 1288 insertions(+)
     create mode 100644 404.html
     create mode 100644 apple-touch-icon-144-precomposed.png
     create mode 100644 categories/index.html
     create mode 100644 categories/index.xml
     create mode 100644 css/custom.css
     create mode 100644 css/hyde.css
     create mode 100644 css/poole.css
     create mode 100644 css/print.css
     create mode 100644 css/syntax.css
     create mode 100644 favicon.png
     create mode 100644 img/hugo.png
     create mode 100644 index.html
     create mode 100644 index.xml
     create mode 100644 sitemap.xml
     create mode 100644 tags/index.html
     create mode 100644 tags/index.xml
    $ git lg
    * ff17216 Publish: 20180529  (HEAD -> master) [Ruiwen Chua 2 seconds ago]
    | * 2f14616 New post: A sample post  (source) [Ruiwen Chua 54 seconds ago]
    |/
    * d9c0248 Initial empty commit  [Ruiwen Chua 2 minutes ago]

Now we see that the rendered site under `public/` has been committed to the `master` branch, while the source material still remain on the `source` branch. Both branches branch from the initial empty commit we made, and both branches can exist simultaneously in the same working environment, thanks to `git worktree`. This suits `hugo`'s publishing workflow pretty well, allowing us to render straight into our `master` branch for committing.




