---
layout: post
title: Deploying Jekyll site to Azure Static Web Apps
date: 2021-02-17 00:00:00 +0100
description: Step-by-step guide to create a Static Web App in Azure and deploy a Jekyll site on it using Github actions
tags: [Azure, Jekyll, Github, Blog] # add tag
toc: false
comments: false
image:
  src: /assets/img/jekyll-island.jpg
  alt: Jekyll Island
---

Azure Static Web Apps is a new technology to deploy static websites using serverless APIs as the backend. This post shows step-by-step how to create a static blog using Jekyll and deploy it using Azure Static Web Apps.

### Prerequisites
- Install [Jekyll](https://jekyllrb.com/docs/installation/)
- An active Azure subscription
- A Github account
- Your preferred [Jekyll theme](https://jekyllthemes.io/)
- Visual Studio Code with Azure Static Web Apps extension installed

### Create your Jekyll App
For this blog I'm using the [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy) so I started by cloning the project from Github locally.
There are some instruction on the theme page to guide you through the process of customizing it to your liking.
Start by customizing `_config.yml` with your information. Posts go into the `_posts` folder in Markdown format, pictures and other media goes into `assets` folder.
You can test your blog locally by running:
```bash
bundle exec jekyll s
```

### Push your application to Github
Once the blog is to your liking, commit the changes to Github:
1. Initialise the Git repository
```
git init
```
2. Create a public repository in Github:
![github-new](/assets/img/jekyll-az-static-web-app/github-new.png)
3. Add the new Github repository as a remote source to your local git:
```
git remote add origin https://github.com/<YOUR_USER_NAME>/<YOUR_REPOSITORY>
```
4. Commit your changes locally:
```
git add .
git commit -m "Initial commit :)"
```
5. Push your local changes to Github:
```
git push
```

### Create your Azure Static Web App
Since Azure Static Web Apps are still in preview, you cannot use ARM templates to deploy your Static Web Apps just yet, leaving you with either deploying it via the Azure Portal or through the VS Code extension.
I'll describe the steps to create the Static Web App using the VS Code extension. Once you have the extension installed click on the `Azure` icon on the left sidebar.
1. Click on the `+` sign to create a new Static Web App
![new-static-web-app](/assets/img/jekyll-az-static-web-app/az-web-app-new.jpg)
2. Choose a name for your app and press enter
![new-static-web-app-name](/assets/img/jekyll-az-static-web-app/az-web-app-new2.jpg)
3. You'll then be presented with the choice of different build presets to select from, choose custom since we aren't gonna use any of the options. Select **Manually enter location**
![new-static-web-app-custorm](/assets/img/jekyll-az-static-web-app/az-web-app-new3.jpg)
4. In the enter location dialog write in `/_site` as that's where Jekyll stores the compiled website
![new-static-web-app](/assets/img/jekyll-az-static-web-app/az-web-app-new4.jpg)
5. The API or Build location is not required for now, press enter until you are presented the option to select which region to deploy the Static Web App. Choose the region closer to where you are.
![new-static-web-app](/assets/img/jekyll-az-static-web-app/az-web-app-new5.jpg)
You'll then be prompted to link your Azure account to your Github account. Follow the steps as instructed. At the end a file will be created in your repository `.github/` folder and the `yaml` file inside that folder will deploy your website to the Static Web App automatically.

### Deploy your App
Because we are deploying a Jekyll to a Static Web App the first run of the action will fail, don't worry.
Run `git pull` to update your local repo to the newly added changes and edit the `.github/workflows/azure-pages-<WORKFLOW_NAME>.yml` file to include some missing settings.
1. Before the line - name: Build And Deploy add the following configuration block:
```yaml
- name: Set up Ruby
  uses: ruby/setup-ruby@v1.59.1
  with:
    ruby-version: 2.6
- name: Install dependencies
  run: bundle install
- name: Jekyll build
  run: jekyll build
```
2. This should be enough changes to make your Jekyll blog compile and be deployed to the static web app in Azure. Commit and push your changes
```
git add .
git commit -m "Your commit message"
git push
```
After the Github action succeeds, you'll be able to access your newly deployed Jekyll blog by going to Azure and checking the Static Web App url:
![new-static-web-app](/assets/img/jekyll-az-static-web-app/az-web-app-new7.jpg)

### Configuring a custom domain
Now, you don't want to access your Jekyll blog using a complex url, you want to map it to the domain you own.
1. To achieve that you need to go to **Custom Domains** menu in the Azure Portal and select `+Add`
2. Enter your domain and click Next
![new-static-web-app](/assets/img/jekyll-az-static-web-app/az-web-app-new6.jpg)
3. Configure a new CNAME DNS record pointing to your Azure Static Web App. This can be done where you bought your domain. 

### Configuring a root domain in an Azure Static Web App
This isn't officially supported as of now. But there's a way if you really need it. Find more info on [this external blog post](https://burkeholland.github.io/posts/static-app-root-domain/)

