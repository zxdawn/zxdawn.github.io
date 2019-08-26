---
title: Migrate blog to Github(HTTPS) and add continuous integration
date: 2018-05-18 17:38:12
tags: ["hexo", "Travis-CI"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Migrate blog to Github(HTTPS) and add continuous integration"
img: "/images/sci-tech/2018-05/TravisCI.png"
---

# Reference

[MARKSZのBlog](https://molunerfinn.com/hexo-travisci-https/#%E6%8C%81%E4%B9%85%E5%8C%96%E6%9E%84%E5%BB%BA)

# Continuous integration

## Set Travis-CI account and Token

1. Sign up [Travis-CI](https://travis-ci.org/) and sync your Git-hub account:

   ![Sign_up](http://ox58se1xg.bkt.clouddn.com/integration/Travis-CI_sign_up.png)

2. Choose your *github.io:

   ![repo](http://ox58se1xg.bkt.clouddn.com/integration/repo.png)

   <!--more-->

3. Set `Token`:

   First step: check `Settings`

   ![setting](http://ox58se1xg.bkt.clouddn.com/integration/setting_github.png)

   Second step: find `Developer settings`

   ![setting2](http://ox58se1xg.bkt.clouddn.com/integration/setting_github2.png)

   Third step: generate your token (you can type any name you like) and check 'repo':

   ![setting3_1](http://ox58se1xg.bkt.clouddn.com/integration/setting_github3.png)

   ![setting3_2](http://ox58se1xg.bkt.clouddn.com/integration/setting_github4.png)

   **Because the token just show once, you must record your token!! Otherwise, you'll need to generate again.**

4. Add Environment Variable

   We can store the generated token as `GH_TOKEN` which will help us write `.travis.yml` file later.

   ![Environment Variable](http://ox58se1xg.bkt.clouddn.com/integration/setting_travis.png)

## Write config file

`.travis.yml` is the config file of `Travis-CI `. You need to generate one and push to the root of your Git-hub repository (*github.io).

Here's the example of `.travis.yml` and you should change according to your own account:

```
language: node_js # 声明环境为node
node_js: stable

# Travis-CI Caching
cache:
  directories:
    - node_modules # 缓存node_modules文件夹

# S: Build Lifecycle
install:
  - npm install -g hexo-cli # 下载依赖

script:
  - hexo g

after_script: # 推送到github的部分
  - cd ./public
  - git init
  - git config user.name "zxdawn"
  - git config user.email "xinzhang1215@gmail.com"
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master # 通过之前存在Travis-CI里的token以及github仓库的地址推送到相应的master分支
# E: Build LifeCycle

branches:
  only:
    - master # 只对master分支构建

env: # 环境变量
 global:
   - GH_REF: github.com/zxdawn/zxdawn.github.io.git # 我的仓库地址
```

# Add HTTPS to blog

I'll explain how to add HTTPS to your blog, if you have your own domain name instead of `*github.io`.

You can add HTTPS easily and freely by `Cloudflare`

## Use custom domain with GitHub Pages

You need to add your domain to GitHub according to [GitHub tutorial](https://help.github.com/articles/using-a-custom-domain-with-github-pages/).

1. Set [custom domain](https://help.github.com/articles/adding-or-removing-a-custom-domain-for-your-github-pages-site/):

  ![Custom domain](http://ox58se1xg.bkt.clouddn.com/integration/setting_github5.png)

2. Create A records that point your custom domain to [GitHub IP](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider):

   ![DNS server](http://ox58se1xg.bkt.clouddn.com/integration/setting_DNSserver.png)

## Sign up [Cloudflare](https://www.cloudflare.com/) account

## Manage DNS

Type `A` points to [Git-hub IP](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider). Tyoe `CNAME` points  `www` subdomain to `apex` domain.

![DNS](http://ox58se1xg.bkt.clouddn.com/integration/setting_cloudflare1.png)

Click 'Next' and change NameServer of your DNS server according to Cloudflare guide.

![NameServer](http://ox58se1xg.bkt.clouddn.com/integration/setting_DNSserver2.png)

## Turn on HTTPS

### Turn on flexible SSL

![SSL](http://ox58se1xg.bkt.clouddn.com/integration/SSL.png)

### Redirect traffic to HTTPS

If others visit my blog by `http://dreambooker.site`, how can I let them visit `https://dreambooker.site` directly?

We can use the function of `Page Rules`:

![Page Rules](http://ox58se1xg.bkt.clouddn.com/integration/redirect.png)

By the way, this method is the redult of communication between browser and server. We can use `HSTS` to make redirect safely and quickly. `HSTS` is under the menu of `Crypto`:

![HSTS](http://ox58se1xg.bkt.clouddn.com/integration/HSTS_on.png)