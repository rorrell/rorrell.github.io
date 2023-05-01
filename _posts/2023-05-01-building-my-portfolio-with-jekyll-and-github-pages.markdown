---
layout: post
title:  "Building My Portfolio with Jekyll and Github Pages"
date:   2023-05-01
categories: tutorial
---
![image tooltip here](/assets/roman-synkevych-wX2L8L-fGeA-unsplash.jpg)

I decided it was time to have a portfolio website.  There were a few reasons for this: 

1. Github only displays six projects
2. My student blog needed to be migrated at some point as it would eventually get shut down
3. It's a little more professional

Obviously there are several options for this.  I looked into multiple ways to do this, but settled on Github Pages with Jekyll because I didn't want to spend a whole of time on this as a backend engineer; I'd rather leave more time for the blog and the projects themselves.  This was advice I saw echoed out on the web as well.

So let's dig in.  What's Jekyll like?  Once you get the [prerequisites installed](https://jekyllrb.com/docs/installation/), it's basically static content in the form of markdown pages.  You have to do some configuration to get it up and running, though.  First you have to run the following command in your target directory:

```
jekyll new .
```

That creates a bunch of files.  Then, in the Gemfile, you have to comment out the following line:

```
# gem "jekyll", "~> 4.3.2"
```

Then uncomment this line:

```
gem "github-pages", group: :jekyll_plugins
```

Next, in _config.yml, I just changed the title, email, description, url, and github_username, and added a linkedin_username.  Finally, I was ready to run the following command in the terminal:

```
bundle exec jekyll serve --livereload
```

However, I ran into an error.  

> cannot load such file -- webrick (LoadError) 

After some searching, I found what I had to do was run this command in the terminal (this is Jekyll 4.3.2, by the way):

```
bundle add webrick
```

Then I repeated the previous command and was able to view my site in a browser at http://127.0.0.1:4000/.  To add pages, I added markdown files at the root, and they were linked at the top of the page.  To add posts, I added markdown files to the _posts folder with the name pattern of YYYY-MM-DD-title.markdown.  I left the index.markdown page in its default state, and it just shows a list of the post titles, which link to the full posts.  I can add something to the _config.yml to make it show the first bit of the posts (show_excerpts: true), but since the first part of all my posts is a large image, that doesn't work out well for me.  Here is the header for this post as an example:

```
---
layout: post
title:  "Building My Portfolio with Jekyll and Github Pages"
date:   2023-05-01
categories: tutorial
---
```

Images, by the way, can be added to posts by placing them in a root-level folder called assets, and doing like so:

```markdown
![image tooltip here](/assets/image.jpg)
```

That link is absolute, so even if you are in the posts folder, it will work.

To have a page of my projects, I added projects.markdown to the root level and added the following:

```
---
layout: page
title: Projects
permalink: /projects/
---
{% raw %}
{% for repo in site.github.public_repositories %}

{% if repo.fork == false %}

## [{{ repo.name }}]({{ repo.html_url }})

{{ repo.description }}

Language: {{ repo.language }}

Last updated: {{ repo.pushed_at | date_to_string }}

{% endif %}

{% endfor %}
{% endraw %}
```

It goes through all the public repositories, and if they aren't forked, it links the name, shows the description, the language, and when it was last pushed (I found the updated_at counted updates to the description and such) as a date string.

By the way, to get that code to display without rendering, I had to enclose it in raw / endraw tags within the curly braces with percent signs.

And that's all I did.  My main issue is that I would like to be able to sort my projects.  If I could have them listed in reverse by date, that would be great, but they are sorted alphabetically.

Next I might look into different themes.  I am currently using the default, minima.  But at least it does the basics of what I need it to do.  It has a blog, and employers can see all the projects I want them to see without have to click on a bunch of things.