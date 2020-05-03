![](/images/blogging/keyboard-1490036-640x480.jpg)

[GitHub Pages](https://help.github.com/en/github/working-with-github-pages) can be used to set up your blog for free with a URL like `username.github.io` (where `username` is your GitHub username). I had to sort through a few combinations of options with GitHub Pages and [Nikola](https://getnikola.com/) settings. I finally picked one that I found optimal and intuitive.

Here are my notes in case it's useful for anyone else. Even if you don't use the Nikola static site generator, the options discussed here for GitHub Pages might still be useful.

<!-- TEASER_END -->

# GitHub Pages Options
A free GitHub Pages site can be **user**-specific or **project**-specific. Here's an outline of what these are and how Nikola can be used with each one.

## User Site
A user site is based on a unique repository name tied to your GitHub username, i.e. you create a public repository called `username.github.io` where `username` is your unique GitHub username. GitHub publishes the pages and makes them visible to the world at `https://username.github.io`.

The problem with the user site is that GitHub will publish the content from the `master` branch of this repository. It is more intuitive to me to use the `master` branch to keep my source files, and have a different branch where I can push the publishable HTML output. Unfortunately, with a user site, there are no such options.

Nikola (I'm using version 8.0.3 while writing this) seems to support the user site approach by default. Nikola's `conf.py` file, where all the user configurations are stored, contains the following defaults:
```python
GITHUB_SOURCE_BRANCH = 'src'
GITHUB_DEPLOY_BRANCH = 'master'
```
What the above means is that when "`nikola github_deploy`" is executed, Nikola will automatically commit and push the `master` branch with the publishable content. It will also automatically create the `src` branch if it doesn't exist. And commit and push the `src` branch with all the (non-`.gitignored`) files in the repository.

Of course, branch names are just conventions, and the names could really be anything. However, not being able to use the `master` branch to keep my source files is unintuitive to me. Note that, while Nikola allows the source and deploy branches to be named as you like, GitHub only serves pages from the `master` branch for a user site.

## Project Site
A project site is a GitHub Pages site that can be published from any public repository you have on GitHub. For a personal blog, let's say you create a public repository called `blog`. Then GitHub will make the pages visible to the world at `https://username.github.io/blog/`.

With a project site, GitHub can be configured to publish from one of three options: the `master` branch, the `docs` folder in the `master` branch, or the `gh-pages` branch. I immediately gravitated towards using the `gh-pages` branch as it allows me to use the `master` branch naturally for all my source files and other settings. Here are the Nikola settings to achieve this:
```python
GITHUB_SOURCE_BRANCH = 'master'
GITHUB_DEPLOY_BRANCH = 'gh-pages'
```
With these settings, Nikola takes care of generating and pushing the `gh-pages` branch.

# Dealing With The Longer Project Site URL
The project site has a longer URL than the user site so that appears to be a disadvantage to using project sites for a blog. However, we can work around that by setting up a redirect from `https://username.github.io/` to `https://username.github.io/blog/`. What this means is that you can tell everyone that your blog is at `https://username.github.io/` and that's all they will have to type in their browser. The redirect automatically takes them to the longer URL.

![](/images/blogging/road-2783679_1280.jpg)

The redirect approach, which I'm using, is based on this wonderful article: [Setup a redirect on Github Pages](https://dev.to/steveblue/setup-a-redirect-on-github-pages-1ok7). You have to create a public repository called `username.github.io` (replacing `username` with your username). In this repository, commit two files to the `master` branch with content as follows (replacing my URLs for the project site-based blog URL with yours):

- [`index.html`](https://github.com/ravi-chandran/ravi-chandran.github.io/blob/master/index.html)
```html
<!DOCTYPE html>
<meta charset="utf-8">
<title>Redirecting to https://ravi-chandran.github.io/blog/</title>
<meta http-equiv="refresh" content="0; URL=https://ravi-chandran.github.io/blog/">
<link rel="canonical" href="https://ravi-chandran.github.io/blog/">
```

The meta tag with `http-equiv="refresh" content="0"` causes the redirect to the specified URL. In addition, the link tag with `rel="canonical"` tells search engines that the two sites have duplicate content.

- [`_config.yml`](https://github.com/ravi-chandran/ravi-chandran.github.io/blob/master/_config.yml)
```
theme: jekyll-theme-cayman
```
A theme has to be set here so that GitHub Pages will actually build the repository's `master` branch as a GitHub Pages site.

# Controlling Commits
There was still one thing that was bugging me. With my Nikola settings for the project site, I liked the fact that Nikola automatically copied the necessary files and committed them to the `gh-pages` branch. But it also committed the branch designated as `GITHUB_SOURCE_BRANCH`. I like leaving configurations at their default state as much as possible so that things are more likely to work in general... However, I wanted to control what and when things get committed, and to do so with meaningful commit messages. Of course, Nikola has a setting for that:
```python
GITHUB_COMMIT_SOURCE = False
```

# Conclusion
With this combination of settings for Nikola and GitHub Pages, anyone can achieve a free blog with a simple URL and still have intuitive configuration control over your blog source files. I hope this was useful for anyone exploring similar options.
