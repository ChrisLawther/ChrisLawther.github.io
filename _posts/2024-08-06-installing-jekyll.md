---
# layout: post
title:  "Installing Jekyll"
date:   2024-08-06 21:55:52 +0100
categories: macos jekyll github
---

GitHub pages is powered by Jekyll. Being able to run it locally makes it much easier to see if your changes are likely to appear as expected. There will still be *some* differences as GitHub overrides/ignores some settings(?)

The [Jekyll](https://jekyllrb.com) docs list just 4 simple steps to get things running locally:


{% highlight zsh %}
gem install bundler jekyll
jekyll new my-awesome-site
cd my-awesome-site
bundle exec jekyll serve
# => Now browse to http://localhost:4000
{% endhighlight %}

If it's not been configured already, a newer version of Ruby installed via `rbenv` is probably a good idea:

{% highlight zsh %}
# Install rbenv
brew install rbenv

# Get (and then follow!) the instructions for integrating it into the shell
rbenv init

# Get a list of the latest stable versions
rbenv install --list

# Install a specific version
rbenv install 3.3.4
{% endhighlight %}

Now, navigate to wherever you intend to create your pages repo, then:
{% highlight zsh %}
# Select a version for the current directory
rbenv local 3.3.4

# Install Jekkyl
gem install bundler jekyll

# Start a new project
jekyll new sitename
cd sitename
{% endhighlight %}
