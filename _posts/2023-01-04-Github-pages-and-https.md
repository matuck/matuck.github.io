---
layout: post
section-type: post
title: Github pages and https
category: Tech
tags: [ 'github', 'webhosting', 'security' ]
---
<img class="floatimageleft borderless" src="/img/githublogo.png"  width="200px" />I have been using github to host my website for sometime now.  I did not have it enabled for https due to the complications of setting this up.  This will go through the complications of setting up https with a custom domain.  I went to Enforce https and I reveceived the messge "Unavailable for your site because your domain is not properly configured to support HTTPS".  So in researching the problem I found several problems that kept me from enabling https on the domain.

The first problem I encounted was my dns a, aaaa, and cname records were not setup correctly.
A records should be set to
 * 185.199.108.153
 * 185.199.109.153
 * 185.199.110.153
 * 185.199.111.153

AAAA records should be set to
 * 2606:50c0:8000::153
 * 2606:50c0:8001::153
 * 2606:50c0:8002::153
 * 2606:50c0:8003::153

CNAME records should point to either @ or <username>.github.io

That tackles the first hurdle.  The 2nd hurdle was getting the ssl certificate generated.  I was getting an invalid certificate when navigating to my domain.  The solution to this was to add a CAA record to my domain.  This record should point to letsecnrypt.org

All information for this was pulled from this article.
https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site