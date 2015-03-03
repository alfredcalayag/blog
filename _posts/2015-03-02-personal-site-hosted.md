---
layout: post
title:  "Learning About Domains and Hosting"
date:   2015-03-02
category: development
tags: aws s3 route53 domain
---

### **TLDR:**
- Static site creation using AWS S3 is quite simple by itself
- AWS Route 53 is also straight-forward
- Setup on Namecheap to point to AWS servers can be confusing.
- Be prepared to wait 48+ hours for your site to be completely working.

### **Overview**
Finally got my personal site hosted.  I decided to go with [Namecheap](http://www.namecheap.com) for the domain and AWS to host.  Specifically, I picked [AWS's S3](http://aws.amazon.com/s3/) service to start my site off as a static page.  It was either that OR creating an EC2 instance with some additional setup.  The latter seemed to be overkill for what I need right now, and I was a little intimidated after hearing that setup on AWS comes with a steep learning curve.  I wanted something simple and quick to get my site up, and I wanted it yesterday!

For the most part, the stories were true.  Namecheap was a user-friendly experience, and AWS S3 was a breeze to upload my site files (at the moment it's just html, css, and js files).  Separately, they were quite simple to setup until I got to the integration part.  **Big Headache!**
I should clarify.  Setting up the root domain site, i.e. "http://rootdomain.com", was simple.  Follow the most basic instructions on both the domain and host sides and you should be fine.

#### **Setup on AWS S3**
As per the [AWS documentation](http://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html), I created a bucket for the "rootdomain.com" and enable web hosting.  On Route 53, I create a Hosted Zone and added an ALIAS record.  All as instructed.  At this point, it worked fine when you visit the http://rootdomain.com.s3-website-us-east-1.amazonaws.com.  But of course, we're not done yet.

#### **Setup on Namecheap**
Here's where you have to make a big decision.  Use Namecheap to handle the nameserver delegation or manually assign the nameservers provided by AWS.  After some googling, I was under the impression that I would have a more reliable experience going with AWS... and here's where more of my uncertainty begins.  I followed the instructions to [Transfer DNS to Webhost](https://www.namecheap.com/support/knowledgebase/article.aspx/767/10/how-can-i-change-the-nameservers-for-my-domain), using the 4 nameservers that are provided in Route 53.  Note that when you use this option, your Namecheap dashboard options changes a bit, and that's to be expected.

At this point, my site works!  I go to http://rootdomain.com and it works.  Hooray!

### **Challenges**
So far so good... sort of.  Until I got to creating subdomains.  And here is where it gets ugly.

In the AWS documentation, you'll find instructions to create a "www" subdomain to re-direct users if they request "www.rootdomain.com" instead of "rootdomain.com".  I attempted to do so for a "www" subdomain and also a "blog.rootdomain.com".

It seemed to be as simple as creating separate buckets for the subdomain names.  The "www" set up as a redirect to the root domain, and the "blog" would have its own files.  And then I created separate hosted zones in Route 53, one each for "www" and "blog" using the same steps s]as for the root domain.

**Important detail here.**  The part that was NOT as clear was the subdomain nameserver setup on Namecheap.  Remember when I made the decision to define the custom (AWS) nameservers for the rootdomain?  Well, now I needed to go back and add the nameservers that were assigned for the "www" and "blog" given by Route 53.  There were 4 each.  Taking a step back, thats 4 nameservers for the root domain + 4 for the "www" + 4 for the "blog".  That's a total of 12, which is the limit for any 1 domain account.  Namecheap was actally limiting me to add 11 myself, and they manually had to add the 12th on on their end (customer service was decent).

All should work now?  After about 10-20 minutes... it sortof did.  All sites worked on my desktop Chrome browser and spotty at times using Safari.  Using my iPhone, using Chrome, Safari, Dolphin, and Opera, the root domain worked, but the redirect and blog subdomains didn't.  For the first 3 days, they would work intermittently on mobile, and I couldn't fully understand why.  Sometimes the page would fail to load, sometimes a blank white page would load.  Here are/were my theories and attempt to debug as to why:

#### **Impatience?**
Namecheaps support pages mentions, "Note that it may take up to 24-48 hours for the changes to take effect worldwide."  That's nice, but I was thinking that 48 hours was a worst-case, not-common situation.  That's quite a long time for me to assume this is the case only to be surprised after 2 days that it doesn't work and the problem was due to something else.  So, I dig further to see what else could be the problem.

#### **Does it Ping?**
Again, while it worked on the desktop browsers just fine, I try pinging each of the sites: rootdomain, www redirect, and blog.  Pinging the rootdomain returns a "Request timeout for icmp_seq ...".  The others either couldn't establish a connection or would ping the same way.  Again, the subdomains had spotty connections.  Still need to research why I'm receiving timeouts.

#### **CORS Issue?**
I double... triple checked that I had the correct CORS configuration on each of the buckets, allowing all headers and all origin using the "*" option.

#### **Bucket Permissions Issue?**
Another low-cost, triple-check.  I used a very standard configuration from the AWS documentation.  No funny business was going here.

#### **Safari Caching Issue**
Relentless googling of "iOS Safari blank white page" lead me to believe the problem could be the mobile browser itself.  Something about the response headers provided from the server (i.e. AWS) was missing a "Vary:Origin" header that the browser was expecting and it the browser would make some incorrect assumptions on the files it should cache, hence rendering a blank page.

I updated my phone's iOS to the latest version, changed my browser settings to allow caching for all sites, and cleared my cookies and history. Still, it didn't fix the problem.

#### **Chrome DevTools Limitations**
I used the iPhone simulator in Chrome's Developer Tools, but didn't observe the same intermittent effects as I did on my actual phone.  The sites would work on desktop and also worked fine on the desktop-chrome-iphone-simulator.  So, I wasn't convinced that the DevTools were going to tell me much... other than it *should* work.

#### **The Solution**
After ripping my hair out the last 3 days, it seemed that all I needed was to wait it out.

I used [Google PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) to run some health-check tests on my sites.  I was happy to found that this tool does a check as a desktop browser and simultaneously as a mobile browser. In the morning of the 3rd and glorious day, it confirmed my observations: loading on desktop, but not loading on mobile.

Retry. Retry. Retry.  I run the same tests several times back-to-back until I got a successful load on the mobile browser.  It was intermittently working.  Weird.

By midday, those tests results became successful every time.  It would now consistently load in the mobile tests.  I checked again on my phone, and it indeed loaded flawlessly with every excessive tap of the refresh button.

Phew!  I guess that solved it?  The only thing I know for sure is the the passing of time from when I registered the AWS nameservers to Namecheap.  The moral of the story is, **Patience is a virtue**.  Yeah, I'm almost certain that this was the problem, but it's still creepy why I couldn't find anyone else experiencing a similar issue.


### **Post Thoughts**

I went through quite a nightmare accusing different areas being the culprit only to find that I needed to wait out process of the AWS nameservers to be registered at Namecheap.  It wasn't all a loss, but rather, it a big gain in knowledge.  I explored several avenues of what the problem could have been, identified the ones that were in my reach as well as the ones I had no control over. I was exposed to several ideas and tools in how to figure out what the problem was.  It was frustrating, but a good learning experience diving into all these different topics in a matter of 3 days.

Was AWS S3 the right choice?  I do want to learn how to use EC2 to create dynamic/scalable sites and apps. Eventually, I will be considering porting my site to an EC2 instance as well as some demo apps I've worked on to share. But for now, the current setup proved to be a solid solution... it eventually did.

If you're considering to use a similar setup as I did, be prepared to do some waiting before your sites becomes reliably live on both desktop and mobile.  Just follow the documentation and give it some time.