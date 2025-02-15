---
layout: blog
title: What's Jekyll?
category: [blog]
tags: [blog]
---

[Jekyll](https://jekyllrb.com) is a static site generator, an open-source tool for creating simple yet powerful websites of all shapes and sizes. 

## To create a Github Page with a Jekylle theme
Assume you have a github account: `https://github.com/yourname`
### Fork Theme
1. Go to [Jekyll-Theme-Hydure](https://github.com/zivong/jekyll-theme-hydure) click [Fork] <br />
2. Click [Settings], **Repository name** enter YourBlogName (Ex. `AI-course`), click [Rename]<br />
3. Delete 2 old branches, keep `master` branch <br />
4. Create new Branch `gh-pages` <br />
5. Click [Settngs], Left Menu `Braches`, **Default branch** set to `gh-pages`
### Add Theme
6. Edit Gemfile to add: <br />
```
theme  "jekyll-theme-hydure"
```
7. Edit _config.yml to add: <br />
```
plugin:
  - jekyll-remote-theme
remote_theme: zivong/jekyll-theme-hydure
```
### Change Cover.jpg
8. Upload a cover.jpg (1920x2880) to **_layouts** <br /> 
9. Modify **_layouts/default.hmtl** to change path of cover.jpg <br />
```
assign cover = site.cover | default: "cover.jpg"
```
### Change Title & Tagline
10. Modify **_config.yml**  <br />
```
- title: AI course
- tagline: Deep Learning Course
```
11. Modify **_data/navigation.yml** to change url<br />
```
- tile Download
  url: https://github.com/rkuo2000/AI-course/archive/refs/heads/master.zip
- title: Github
  url: https://github.com/rkuo2000/AI-course
```
12. Modify **_data/social.yml** to change url<br />
```
- title: Email
  url: mailto:yourname@gmail.com
  icon: fas fa-envelope
- title: FaceBook
  url: https://facebook.com/yourname
  icon: fab fa-facebook
- title: Twitter
  url: https://twitter.com/yourname
  icon: fab fa-twitter
- title: GitHub
  url: https://github.com/yourname
  icon: fab fa-github
```
### Edit Posts
13. Modify **_posts/*.md**, upload or remove .md) <br />
    (posts display latest dated .md on the top) <br />
```
    2021-09-23-DeepLearning-intro.md
    2021-09-23-AI-intro.md
    2021-03-01-whats-jekyll.md
```
14. Modify **_layouts/default.html** for hiding side-header with a checkbox <br />
    see [_layouts/default.html](https://github.com/rkuo2000/AI-course/blob/gh-pages/_layouts/default.html) for detail
### Open Github Page    
    Click [https://yourname.github.io/YourBlogName](https://rkuo2000.github.io/AI-course) <br />


