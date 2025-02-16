---
draft: false

title: Github pages, Hugo and Cloudflare
description: Setting up a simple site with Hugo, Github Pages, and Cloudflare

date: 2021-06-30T20:45:00+01:00
tags: 
    - hugo
    - cloudflare
    - github pages
categories:
    - development
    - github pages
    - hugo
---
So I thought it was about time I started one of these things, collected some thoughts and put some ramblings out into the ether.

First things first - lets get started with how I setup this page!

## GitHub Pages
Docs are [here](https://pages.github.com/)
Basically we need to create a repo called: `drtbz\drtbz.github.io` (or you know.. your username..)
In there, we need a CNAME file with the sole line: `drtbz.com`
And chuck in a `README.md` with some text to check your site works.
That's it, for now!

## Cloudflare
I'm using cloudflare as both registrar and CDN/WAF for this project. You don't have to, but they have some cool stuff, and it's free apart from the registrar bit.

As we're using an apex domain with Github Pages, we've gotta do the following:
 - 4 x A records to drtbz.com with IP's [185.199.108.153,185.199.109.153,185.199.110.153,185.199.111.153]
 - 1 x CNAME for `www` to `drtbz.github.io`
 - A whole bunch of SSL settings:
    - Set to `Full (Strict) mode` because we can.
    - Always use HTTPS
    - Enable HSTS with everything on and at least 6 months Max Age Header
    - Minimum TLS 1.2
    - Opportunistic Encryption ON
    - TLS 1.3 ON
    - Automatic HTTPS ReWrites ON
    - Certificate Transparency Monitoring - Optional, but kinda cool.

That's pretty much it from the cloudflare perspective. Very much set and forget, and just let it do it's thing!

Next up - we're going to setup HUGO, and then do some stuff with Github Actions to tie it all together.

## Hugo
Hugo is super easy to setup. It can be found [here](https://gohugo.io/getting-started/installing/) and the best part is that is written in go - we don't need to install any thing extra to make it run. Installing [go](https://golang.org/doc/install) could be handy if you want to extend in the future though. I used the [windows](https://gohugo.io/getting-started/installing/#windows) hugo installer steps.

Once it's all extracted to a place of your choosing - I recommend you get it into your $ENV:Path, makes life loads easier!

The [QuickStart](https://gohugo.io/getting-started/quick-start/) is also pretty good for getting familar with it. Just bear in mind that guide is aimed at MacOS so some commands need tweaking.

Long story short - clone your version of `drtbz\drtbz.github.io` to your machine. `CD` there, and then run the following commands:

```
#Powershell
hugo new site drtbz.com
hugo mod init drtbz/drtbz.github.io
hugo new posts/some-kind-of-post-name.md
mkdir static
```
Into that `static` folder - add a copy of the CNAME file we setup earlier.

now if it doesn't exist already create a `config.yaml` in the root of your repo. There may already be a `config.toml` there - feel free to convert it to YAML (my preference). This is where we'll do a bunch of config for our site, including menu setup, google analytics (if your that way inclined), icons, and all sorts of funky stuff.


I've chosen to use [PaperMod](https://github.com/adityatelange/hugo-PaperMod) as my theme. The setup guide for that is pretty good - or feel free to checkout my [config.yml](https://github.com/drtbz/drtbz.github.io/blob/main/config.yaml)

Once you've tweaked the config.yml. That is literally all we need to get going! Have a look in `.\Content\posts\` for the `.md` file from the last step. There's a bunch of stuff there. For example the post your reading looks like this:
```
---
draft: false

title: Github pages, Hugo and Cloudflare
description: Setting up a simple site with Hugo, Github Pages, and Cloudflare

date: 2021-06-29T21:00:00+01:00
tags: 
    - hugo
    - cloudflare
    - github pages
categories:
    - development
    - github pages
    - hugo
slug: 20210630-Hugo-GithubPages-Cloudflare-01
---
```
Most of this is self explanatory.

Now you can either use Hugo locally to run a little webserver, or you can setup github environments and actions to do all the generation on a push to master.

## GitHub, Pages and Actions
First up - jump into your version of `drtbz.github.io`. Create a quick branch called `gh-pages`

now find the `settings` and then `environments`. 

Create a new one called `hugo`. In the configuration you can configure `protection rules` as you please, but you do need to select the `deployment branch` to `main` or whichever one YOU are going to push code to that you want to make live as your page - either directly, or via PR from a test branch.

Same kinda deal for the `github-pages` environment except the chosen branch here is `gh-pages` that we created earlier. The reason for this is that when we run the `action` on commit - the hugo builder will deploy the static content to the `gh-pages` branch, allowing us to keep the raw "code" we build, and the actual site data seperated.

### Action!
There's a couple of ways to set this up, but the simplest is to navigate to the `actions` tab in the repo, and hit `new workflow`, then the `setup a workflow yourself` link.

I chose the filename [gh-pages.yml](https://github.com/drtbz/drtbz.github.io/blob/main/.github/workflows/gh-pages.yml)

And then add the following:
```
name: hugo -> pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    environment: hugo
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: recursive  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```
This is configured to use the `hugo` environment, so it will only run on pushes to `master`. It will then generate your static site, and place it in the `gh-pages` branch, that GitHub Pages is looking at to do it's magic.

Easy? Let me know!






