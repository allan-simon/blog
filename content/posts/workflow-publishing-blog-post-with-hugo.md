+++
date = "2015-02-20T15:03:36+08:00"
draft = true
title = "Workflow publishing blog post with hugo"

+++

I've added some functions in my shell to make it even more convenient
to write/edit/publish articles with `hugo` (just a self reminder)

```
export BLOG_ROOT=~/git/perso/blog
export BLOG_THEME=hyde

function new_blog_post {
    cd $BLOG_ROOT
    LAST_BLOG_POST=$(hugo new posts/$1.md | cut -d " " -f 1) ;
    export LAST_BLOG_POST ;
    $EDITOR $LAST_BLOG_POST
    cd -
}

function publish_blog {
    cd $BLOG_ROOT;
    hugo --theme="$BLOG_THEME" --buildDrafts && \
    git add -A &&  \
    git commit -m "Updating site" && \
    git push origin master && git subtree push --prefix=public && \
    git@github.com:allan-simon/blog.git gh-pages --squash  && \
    echo "published"
    cd $BLOG_ROOT
}

function edit_last_blog_post {
    $EDITOR $LAST_BLOG_POST
}
```

