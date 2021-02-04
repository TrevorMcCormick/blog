---
title: "👨‍💻 How To Build a Serverless Hugo Blog on AWS for $0.51 per month"
date: 2021-02-02T11:20:22-04:00
draft: true
tags: [AWS, How-To, Hugo]
---

*This post is in draft mode and is not done yet!*

Below is an AWS architecture diagram I created using [diagrams.net](diagrams.net), the free version of draw.io. This diagram visualizes how my website runs for just $0.51 a month (exc;uding the $12/year domain name registration). While making this, I was a little fuzzy on how Route53 actually worked, so I included some additional detail on how it interfaces with an example ISP. Here are the main steps I'll talk about in this diagram:
1. **Route53** -- registering a domain, validating it, and routing traffic to Cloudfront
2. **Cloudfront** -- mapping your S3 bucket endpoint
3. **S3** -- hosting your blog as a static website
4. Your Code -- generating a Hugo blog 
5. **GitHub** -- hosting your code
6. **CodeBuild** -- building your Hugo site

![](/page/images/trmccormick.com.jpg#align-center)
 
## Route53
I pay $0.50 per month for one hosted zone on [Route53](https://aws.amazon.com/route53/). AWS actually created this hosted zone for me when I purchased my domain through Route53. Once you purchase a domain, you need to obtain the SSL/TLS certificate through ACM to identify the site over the Internet. [Here is exactly how you do that](https://aws.amazon.com/blogs/security/easier-certificate-validation-using-dns-with-aws-certificate-manager/). So far you should have 2 DNS records set up from Route53 (NS, SOA) and 1 or more records set up from ACM (CNAMEs). Later you'll add an A record to route traffic to your Cloudfront distribution for each CNAME record.

If you're interested in the piece of the diagram focused on connecting to the Internet and the relationship between ISPs and Route53, I learned a lot from [How internet traffic is routed to your website or web application](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/welcome-dns-service.html). 

## Cloudfront

## S3
## Your Code
## GitHub
## CodeBuild