---
date: "2020-04-05"
tags: ["web", "productivity", "hugo"]
title: "Configuring Staticman Comments with Hugo"
---

UPDATE: Fix broken links to staticman's partial, CSS and JS files.

I wanted to add comments to my blog, and Disqus seemed like a good option as the [theme](https://github.com/zwbetz-gh/cupper-hugo-theme) I'm using supports it out of the box. However, as things stand, I am happy with a solution that doesn't require storing people's data in third party databases and doesn't add [ads and unnecessary tracking scripts](https://replyable.com/2017/03/disqus-is-your-data-worth-trading-for-convenience/) that could make the reading experience slower or cluttered.

After searching for open-source/ethical comment suppliers, I found out about [Staticman](https://staticman.net/), and I am giving it a try since it integrates with Hugo blogs, it uses a git repository to store and triage comments, it's been around since 2015, and has good documentation. I just had to work around some constraints. In essence, you need to deploy your own instance of Staticman to Heroku as the official Staticman API hits its quota frequently (Heroku's free tier is enough tho'), I wanted to keep this blog's comments on a separate repository, and I am using Staticman API V2 since everything is hosted on GitHub (V3 supports other providers like Gitlab). 

This [post](https://petersen.pro/blog/staticman-comments-in-separate-repository/) by Arne Petersen was of great help to put everything together. After some tweaks, my deployment works like this:

- I'm only collecting people's names and comments
- I'm using reCaptcha 2 to avoid spam
- I only load reCaptcha's JS script when you click the "Show Comments" button
- I accept/reject comments using pull requests.
- After accepting a comment, my blog is re-build and published automatically in Netlify using webhooks
- After someone submits a comment, they get redirected to the original blog post with a message explaining their comment will go live after approval. No AJAX or popups are required, and you can try it leaving a comment!

And the instructions:

1. Create a repository for your comments in your `main` GitHub account; we will call it `blog_comments`
2. Create a secondary GitHub account; we will call it `account2`. This is for security reasons as Arne pointed out, you are creating a Personal Access Token and keeping it in your Heroku instance which could give anyone who gets hold of it full access to your GitHub account.
3. Create a Personal Access Token in `account2` at [https://github.com/settings/tokens](https://github.com/settings/tokens). Save it because you can only see it once, and you will need it in a bit.
4. Invite `account2` as a collaborator to `blog_comments` going to `https://github.com/YOUR_MAIN_GITHUB_ACCOUNT/blog_comments/settings/access`
5. Deploy Staticman to Heroku using the purple button in the project's [README](https://github.com/eduardoboucas/staticman) (make sure it's in the master branch)
6. Create a private key for Staticman (you can do this in your Heroku instance going to "More" -> "Run console"): `openssl genrsa -out key.pem`
7. Add the following three Config vars to your Heroku instance in `https://dashboard.heroku.com/apps/YOUR_INSTANCE_NAME/settings`:
    ```yaml
    NODE_ENV production
    GITHUB_TOKEN "YOUR PERSONAL ACCESS TOKEN"
    RSA_PRIVATE_KEY "CONTENT OF key.pem"
    ```
8. If you want to use reCaptcha to avoid spam, do the following:
 1. Register your blog [here](https://www.google.com/recaptcha/admin). You can add `localhost` to the domain list to be able to test everything in your local machine. Save your `siteKey` and `secret`
 2. Encrypt your reCaptcha `secret` obtained before by querying your Heroku instance in this URL: `https://YOUR_HEROKU_APP_NAME.herokuapp.com/v2/encrypt/YOUR_UNENCRYPTED_RECAPTCHA_SECRET`
9. Add [this partial](https://gist.github.com/JulioV/c1386fde8920406f3871666bf059d1a3) to your blog and call `{{ partial "staticman.html" . }}` where you want to load your comments.
10. Add these [CSS rules](https://gist.github.com/JulioV/5e0297961e4425054ec94c44c880fc70) to your blog.
11. Add this [JS script](https://gist.github.com/JulioV/8f3bfd3113764fc9c66726a12d651820) to your **partials**
12. Add the following lines to the `params` list in your Hugo blog's `config.yaml`
    ```yaml
    staticman: 
    api: https://<YOUR_HEROKU_APP_NAME>.herokuapp.com/v2/entry/YOUR_MAIN_GITHUB_ACCOUNT/blog_comments/master/comments
    recaptcha:
    sitekey: "YOUR RECAPTCHA KEY"
    secret: "YOUR ENCRYPTED RECAPTCHA SECRET"
    ```
13. Add your [Staticman configuration file](https://staticman.net/docs/configuration), `staticman.yaml`, to the root of `blog_comments`. You can use the one below or [this other one](https://raw.githubusercontent.com/eduardoboucas/staticman/master/staticman.sample.yml) as a reference if you want to collect more data like emails or personal websites.
    ```yaml
    comments:
    allowedFields: ["name", "comment"]
    branch: "master"
    commitMessage: "New comment in {options.slug}"
    filename: "comment-{@timestamp}"
    format: "yaml"
    generatedFields:
    date:
    type: date
    options:
    format: "iso8601"
    moderation: true
    name: "YOUR SITES NAME"
    path: "{options.slug}"
    requiredFields: ["name", "comment"]
    transforms:
    email: md5
    // Delete this if you do not want to use reCaptcha
    reCaptcha: 
    enabled: true
    siteKey: "YOUR RECAPTCHA KEY"
    secret: "YOUR ENCRYPTED RECAPTCHA SECRET"
    ```
14. Add your `blog_comments` repo as a submodule to your main repo in the `data/comments` folder: `git submodule add https://github.com/YOUR_MAIN_GITHUB_ACCOUNT/blog-comments.git data/comments`
15. I use Netlify to publish my blog, so I had to modify my Netlify build command to pull the latest version of `blog_comments` to render any new comments. You can do this using Netlify's website or by adding a `netlify.toml` file to the root of your blog repo with the following lines:
    ```toml
    [build]
    publish = "public"
    command = "git submodule update --remote data/comments && hugo --gc --minify"

    [context.production.environment]
    HUGO_VERSION = "v0.68.3"
    HUGO_ENV = "production"
    HUGO_ENABLEGITINFO = "true"
    ```
16. Every time someone comments on a blog post, Staticman creates a new branch and a Pull Request (PR) in `blog_comments` which you can accept or reject to publish it or not. Branches will start to pile up, so, for those PRs you reject, you have to delete their branches using GitHub's UI manually. Still, for those PRs you accept, GitHub can automatically delete them by activating [this feature](https://help.github.com/en/github/administering-a-repository/managing-the-automatic-deletion-of-branches).
17. At this point, you can submit your first comment from your computer or, commit everything to GitHub and do it online.
18. *Optional*. If you want to avoid triggering a new Netlify build manually every time you accept a comment, you can automatize it by using [Integromat's webhooks](https://www.integromat.com). You could also use Zappier, but you have to switch to their paid tier.
     1. Got to Netlify's `Build hooks` section in `https://app.netlify.com/sites/YOUR_NETLIFY_DOMAIN/settings/deploys#build-hooks` and click on `Add build hook`. Save the generated URL
     2. Create a new Scenario in Integromat
     3. Add a `Custom Webhook trigger`. Inside, add a new `Webhook` and copy its URL. Click on `Determine data structure`
     4. Go to `https://github.com/YOUR_MAIN_GITHUB_ACCOUNT/blog_comments/settings/hooks`. Click on `Add webhook`, in `Payload URL` add the URL of the Integromat `Custom Webhook trigger`, in `Content type` select `application/json`, and under `Let me select individual events` check `Pull requests`. Click on `Add Webhook`.
     5. Submit a comment in your blog, so Staticman creates a new Pull Request in `blog_comments`, and Integromat infers its content. You should see a confirmation message in the `Custom Webhook trigger`. 
     6. Add a `HTTP action` in Integromat. Connect this to the `Custom Webhook trigger`
     7. In the connection between the `HTTP action` and the `Custom Webhook trigger`, add two conditions joined by an `AND` operator: `action` = `closed` and `pul_request: merged` = `true`. They should be autocompleted if Integromat was able to infer the PR's content
     8. Click in the `HTTP action`, add the Netlify hook's URL you got earlier to the action's `URL` field, and change its `Method` to `POST`
     9.  Turn the scenario ON using the switch at the bottom left and set `Schedule setting` to `Immediatly`
     10. From now on, the scenario should trigger a Netlify build every time you accept a Staticman's Pull Request

Feel free to leave a comment if you have issues or questions!
