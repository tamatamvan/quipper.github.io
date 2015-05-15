Quipper Tech Team Blog
======================

The Quipper Tech Team Blog is written in [Jekyll](http://jekyllrb.com/) and uses the [Mediator](http://jekyllthemes.org/themes/mediator/) theme with a few tweaks here and there.

### Adding posts

Adding posts to the blog is really easy! You can even do it right here in Github.  
First, navigate to the `posts` folder and click the `+` to add a new file.

![image](https://cloud.githubusercontent.com/assets/1628558/7653142/898d0902-fb0b-11e4-81a5-d6992f874ed3.png)

Give your file a name that conforms to `YYYY-MM-DD-filename.markdown` then start typing a post in Markdown format.

You will need to start your post with some [YAML Front Matter](http://jekyllrb.com/docs/frontmatter/) which describes some important things about your post. It's recommended that you at least supply the following:

```
---
layout:   post
title:    "TITLE OF YOUR ARTICLE"
date:     YYYY-MM-DD HH:MM:SS
author:   YOUR ALIAS (more on this later)
comments: true (if you want to include disqus comments, which is highly recommended)
---
```

Once you have finished writing your post, it is recommended that you submit it as a Pull Request for another to proof-read.

![image](https://cloud.githubusercontent.com/assets/1628558/7653286/7463b458-fb0c-11e4-8d9a-a109d9061719.png)

When the Pull Request is merged, your post will be immediately available at http://quipper.github.io !!

### Images

If you want to add images, you can upload them to the `assets/article_images` folder, but it may be easier to just use Github's in-built image hosting. The trick is to find any Github issue, go to the new comment box, and then Cmd+V to upload a new image. Copy the markdown that Github gives you, and paste that into your article. Voilà!

### Syntax highlighting

If you write code, use the following to highlight it.

```
{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
{% endhighlight %}
```

### Authors

If this is your first time writing a post, you will need to add yourself as an author to the site.

Find and edit the `_config.yml` file that is in the root folder of this repository, and scroll down to the `authors:` section. All you then need to do is add an alias for yourself and give yourself a name and image. Make a pull request for the config changes and you will be registered as an author.

Every post should be identified with an author. (After all, give credit where credit is due!) However if a post has no author attribute then it will simply be credited to Quipper.
