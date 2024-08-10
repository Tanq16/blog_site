---
title: Hosting a Website using GitHub Pages and Jekyll
date: 2023-09-09 15:52:00 -0600
categories: [Computers and Fun]
tags: [github-pages,github,jekyll,deployment]
image:
  path: /assets/img/covers/jekyll-cover.jpeg
  alt: Jekyll Artwork
pin: true
---

## Covering Bases

[GitHub Pages](https://pages.github.com/) is a straightforward method to deploy a website right from a GitHub repository, especially for those with computer science skills. In a very simplistic way, GitHub pages can simply use the repository root as the webroot and serve a website from there. It can also serve from a specific branch.

The deployment of pages can be executed via GitHub Actions (GHA) or directly from a GitHub branch. Of course, there are two basic ways of deploying sites in GitHub: we write normal website code and push it to a branch for GitHub to serve from, and we use static site generators like Jekyll. Check out [GitHub documentation](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages) for more details.

The limitations of GitHub pages to keep in mind are &rarr;

- GitHub only serves static content, so there's no particular server-side processing through PHP, node backend, etc.
- Site content from the repos can be a maximum of 1 GB, meaning videos, music, photos, etc., are typically best placed elsewhere and not inside the repo.

Read the other limitations as well, but those are the key ones. Based on these limitations and the type of people who *generally* use GitHub pages, such sites are best used for portfolio websites, blogs, and documentation sites. Of course, there are other uses, but these sites are easy to maintain and best suited for version-controlled code.

## Custom Domains

GitHub also (obviously) allows the use of custom domains. Put simply, these are several domain-related scenarios for GitHub pages &rarr;

- **No domain, primary site** &rarr; The best way to make a portfolio without a custom domain is to deploy it as `<username>.github.io`. To do this, one must make a repo with the name `<username>.github.io` and deploy their site there. The site is then accessible at `https://<username>.github.io`.
- **Primary site with domain** &rarr; GitHub pages allows setting a custom domain from the repo settings page. It's pretty straightforward, and after setting it up, the site is accessible at the domain. Example - [https://tanishq.page](https://tanishq.page) is deployed similarly. Setting up a domain also requires some other settings from the domain registrar's portal - specifically, linking the domain to GitHub's server - which can be done by following this [GitHub documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain).
- **Subdirectory site, no domain** rarr; If a repository other than `<username>.github.io` is used to serve a GitHub pages site, then the URL for reaching it will be `https://<username>.github.io/<repo-name>`. The issue is that the same is not possible with a domain, i.e., you can't deploy a new repository as a subdirectory for the primary site.
- **Subdomain site, custom domain** &rarr; A completely separate repo can be used as a subdomain site for the primary domain by setting the appropriate subdomain in the settings. Given the primary domain is already set up with a GitHub pages site, this step is straightforward and allows hosting from a different repo. See also [GitHub's documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages).
- **Subdirectory site, custom domain** &rarr; As stated above, doing this isn't possible from a separate repo; therefore, the site resources must be deployed from the same directory as the primary domain but inside a subfolder. Then, visiting that domain's subdirectory will serve that site. Example - this blog site is deployed in the same manner [https://tanishq.page/blog](https://tanishq.page/blog/).

>The subdomain vs. subdirectory debate is a split decision across the internet. I'm not an SEO guru by any means, but the consensus seems to be that if you're an established business, it's okay to have a subdomain site, specifically for things like regional sites or alternate offerings, i.e., all cases where you want a different keyword association for the subdomain site than the primary domain. However, it's generally better for individuals like myself to have a subdirectory since all keywords link to the same domain and boost SEO.
>
>Again, the guidance could be clearer since we also see established organizations like Google shifting some sites from a subdomain to a subdirectory. Many organizations also make the subdomain a pointer to the subdirectory for extra audience reach. All that to say - pick your poison and make an informed decision.
{: .prompt-tip }

## Using a Static Site Generator - Jekyll

This section is one of the main reasons behind this blog because there are fewer descriptive public instructions on Jekyll. I'm also focussing primarily on Jekyll since it's pretty widely used and also the recommended generator to use with GitHub. I also don't want to discuss deploying generic HTML/CSS/JS sites since all that is reasonably straightforward. For example, [my site](https://tanishq.page) is a customized version of a theme from [HTML5UP](https://html5up.net) that I've slapped onto a GitHub repo. So, I'll focus on Jekyll stuff, which is useful for creating a blog like mine.

I choose Jekyll (or static site generators) because we - the people of CS - like our documentation, notes, blogs, information, etc., to stay in a simple and extensible format, Markdown. Jekyll takes markdown content and makes a beautiful website out of it (of course it's gorgeous because of the work various people put behind designing Jekyll themes). A huge collection of themes is available to browse at [JekyllThemes](https://jekyllthemes.io). My blog site (the one you're reading) is based on the [Chirpy](https://chirpy.cotes.page) theme (do check this project out; it's amazing).

### Setup and Serve a Jekyll Site

>I'll stick to instructions for Ubuntu, but just translate these for other distributions of linux or simply deploy it in a container :)
{: .prompt-info }

The first step is the prerequisites. Here is a one-liner `apt` command for installing all dependencies &rarr;

```bash
sudo apt update -y && \
sudo apt install ruby-full build-essential zlib1g-dev g++ gcc make -y
```

Next comes the installation of Jekyll itself &rarr;

```bash
echo 'export GEM_HOME="$HOME/gems"' && \
echo 'export PATH="$HOME/gems/bin:$PATH"' && \
gem install jekyll bundler
```

With the prerequisites out of the picture, simply clone or download your theme, make your changes, and then from the directory, run the following &rarr;

```bash
bundle exec jekyll b -d servedir
```

Now the site is built and is available for serving via the `servedir` directory, i.e., the directory that contains the HTML/CSS/JS for the site. In my experience, it's best to generate this folder and move it around to serve via Apache, GitHub pages, or even plain old `php -S 0.0.0.0:8000`.

### Deploying Jekyll-based Site in a Repo

Usually, all Jekyll-themed templates come with a GitHub action that builds the site using Jekyll (similar to the previous subsection) and deploys the built artifacts via GitHub pages. The associated GHA handles all the steps, including the deployment. For example, check out the workflow by Chirpy creators [here](https://github.com/cotes2020/chirpy-starter/blob/main/.github/workflows/pages-deploy.yml).

If the Jekyll template doesn't have a GHA for deployment, the only option is to make a custom GHA with the steps outined under "Setup and Serve a Jekyll Site". Then, use the created artifacts to manually upload directly to a repo branch or automatically through the action. Then, the repo can be set to serve a site via GitHub pages from that branch. In the case of a subdirectory, neither the generally available actions nor a custom one can deploy a site from one repo to another (from the subdirectory or subdomain repo to the primary repo). So, if we can dissect that problem, the resulting solutions can also be used for deploying to a single repo.

### Deploying a Jekyll Site as a Subdirectory

Static sites can be generated as artifacts and uploaded to a branch or the same repo. Then those artifacts can be used on the repo where the subdirectory needs to be deployed. My site is built the same way; check out [main](https://github.com/tanq16/tanq16.github.io) and [blog](https://github.com/tanq16/blog_site). There are multiple solutions to solving this problem, such as using git submodules, subtrees, or even manually doing the uploading from one repo to another. But obviously, we want to automate this.

I used a GitHub Action solution that automatically pulls a generated site from my blog repo to my main site repo. I'm simply using `git` to clone my blog site and commit only the generated site part. This is possible since my repos are public (GitHub pages can only work on public repos in the free plan). For private repos, similar solutions must use GitHub personal access tokens (PATs). In my case, I'm relying on my repos being public and GHAs having their token for the repo they're defined for.

### Designing the GHAs

For the blog site, I'm using a customized version of Chirpy's GHA to build the site and push the generated content to the same repo &rarr;

```bash
name: "Build Site"
on:
  push:
    branches:
      - main
      - master

  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 1

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3

      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3
          bundler-cache: true

      - name: Build site
        run: |
          rm -rf blog
          bundle exec jekyll b -d blog
        env:
          JEKYLL_ENV: "production"

      - name: Upload site
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "tanishq-github-actions[bot]@users.noreply.github.com"
          git add -A
          git commit -m "[bot] build blog"
          git push
```

Basically, the action is manually executable due to the `workflow_dispatch` being present and automatically executable through changes in the blog repo code. Ultimately, the action commits the generated files in the `blog` directory to the same repo. This means that if I locally run a `php -S 0.0.0.0:8000` from the repo root, the site should be working at `localhost:8000/blog/`.

>For the site generator to work, the base URL for specifying paths for CSS and JS assets must be the `blog` directory, not the repo root. Setting this can be tricky based on the chosen Jekyll theme, as some might hardcode it to be the root, but others may have an option to specify the path. Chirpy has such an option for the record (check the `_config.yml` file for an example).
{: .prompt-warning }

Over on the main website repo, I have the following GitHub action &rarr;

```bash
name: "Integrate Blog"
on:
  push:
    branches:
      - main
      - master

  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Pull blog
        run: |
          rm -rf blog
          git clone --depth=1 https://github.com/tanq16/blog_site
          mv ./blog_site/blog ./blog
          rm -rf blog_site

      - name: Deploy blog
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "tanishq-github-actions[bot]@users.noreply.github.com"
          git add -A
          git commit -m "auto-add blog"
          git push
```

I clone the directory generated earlier on the other repo and commit it to the primary repo. This action can also be launched on demand.

Again, this setup works for me and is ***one*** possible way to get the job done. It's also a naive solution but worth setting up for simplicity. Otherwise, git submodules might be another good solution to try.

## Jekyll is Cool!

The intent behind writing this blog was to share some information on deploying sites via Jekyll and GitHub pages in various scenarios. This helps people search about and debug Jekyll and GitHub pages, especially for hosting subdirectories. Jekyll is fantastic and allows building sites effortlessly, so I highly recommend it! Cheers!
