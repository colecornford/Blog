---
date: "2021-09-09"
tags: ["AppSec", "Code"]
title: "Reflections on OWASP Top10 2021"
---

Today the newest iteration of the [OWASP Top 10](https://owasp.org/Top10/A00_2021_Introduction/) was released. The OWASP Foundation is dedicated to improving web application security. Tools, Educational Material, Advocacy, and more all help make the world more secure. I've always found OWASP content to be insightful and helpful throughout my career and know I'm standing on the shoulders of giants as I write my crappy blog posts.

Today, The OWASP Top 10 2021 edition was released. The OWASP Top 10 is the Flagship project of the foundation. It's a list of the Top 10 most prevalent problems in web application security in that year. In prior years, it's been mostly generated from statistical analysis of tool outputs. This time around the methodology has changed a bit to include some survey data too.

> This installment of the Top 10 is more data-driven than ever but not blindly data-driven. We selected eight of the ten categories from contributed data and two categories from an industry survey at a high level. - OWASP Foundation

I think these changes are great to see as most folk I speak with are encountering issues that are not easy to pick up with automated tools, and web-app security is so much more than running SAST, DAST, and fuzzing. Anyway, the 2021 Top10.

1. **Broken Access Control**
2. **Cryptographic Failures**
3. **Injection**
4. **Insecure Design**
5. **Security Misconfiguration**
6. **Vulnerable and Outdated Components**
7. **Identification and Authentication Failures**
8. **Software and Data Integrity Failures**
9. **Security Logging and Monitoring Failures**
10. **Server-Side Request Forgery**

I thought I'd write some quick thoughts about each category compared to last Top10.

# 1. Broken Access Control

Happy to see this as #1 in 2021. Software was simple back in the day...

![Simple](../../img/posts/2021-09/SimpleApp.JPG)

Eventually executives realised that having apps crash during christmas rushes or having banking fail for thousands of customers was bad for business. So, engineers added redundancy.

![Oh no](../../img/posts/2021-09/RedundancyApp.JPG)

Fast forward twenty odd years and we're now at the stage where apps need to meet a tonne of non-functional requirements before going anywhere near production. Performance, Scalability, Resilience, Security, Maintainability, Reliability, anythingability. You name the NFR, it's probably popping up in conversations down Palo Alto way every minute.

![Here we are](../../img/posts/2021-09/ModernApps.JPG)

Apps are now built using resume-driven-development and over-engineered as fuck. I'm not necessarily seeing 'Does this do what the customer wants' as much as 'Why NOT Kubernetes?'.

Anyway, this complexity makes it hard to do proper access control. Take the above app, draw lines between every single service like a network mesh diagram and you'll understand why. 

If you're asking any of the following, welcome to 2021:

* Who/What should be able to run this function? When?
* Should this service be able to communicate to other services? In what way?
* How do I verify this service?
* Why are they using JWTs for everything?

So off history lane, in 2021 I was fully expecting to see Access Control near the top of the list.

# 2. Cryptographic Failures

I like this being higher than injection. Mostly because I'm sick of seeing developers committing secrets to github `wJalrXUtnFEMI/K7MDENG/bsStoPiTtYAsokAY`. 

Look, can people please stop throwing their secrets into code and then turning their brain off and writing.

```cmd
git add .
git commit -m codefuckbug
git push
```

Use git ignore. Use Trufflehog. Use the tools provided by the SCM companies to stop putting secrets in your source.

Anyway, moving past that rant, engineers know security is important and associate crypto with that. Great start, but I find a lot of crypto problems arise from:

* Engineers lacking business context
* Architectural constraints
* Not being aware of or taught about security yet
* Get it done by Tuesday ya **GRONK**

So I think having cryptography as #2 is a good callout to try and get that at the front of peoples minds as it's needed as part of design, not an afterthought.

However, not a fan of the name. Cryptographic Failures implies to me that the cryptographic system failed. Not that people forgot about crypto, used weak algorithms, or misunderstood what to protect. In each of those cases, the Cryptography didn't fail, the engineer and business did. I think a more appropriate name might be _Insufficient Cryptographic Protection_. It's clearer that cryptography is needed, and that there is an expectation to reach a certain level of maturity.

# 3. Injection

This consolidation is an important change, and the content is now tech-agnostic. Validate your input, reject known bad, parameterize and escape user input, don't concatenate, etc. It's the same fixes whether you use SQL, JSON, GraphQL, ColdFusion, Elixir, or anything else. Previous cheatsheets extended for hundreds of pages to try and cover all tech. This lead to it be completely unconsumable. Check out the OWASP Code Review guide as an example. ;)

I also think the downgrade is appropriate. 2021 Browsers and application frameworks have built-in safeguards. Injection bug-classes are being squashed. DOMPurify for Output Sanitisation, Cookie Attributes to make session hijacking hard, Web Security Headers to stop content downloads and information leaks. Heck, React did enough to force XSS off the list entirely, while SameSite helped murder CSRF. In the API-world we're seeing allowlist specs and with cloud services, native protective controls like WAFs being adopted or on by default.

So while there has been a heap of great work here, I think there is still plenty of room for injection attacks by contextually understanding the application and injecting content to abuse the design. Which leads to number 4.

# 4. Insecure Design

I personally love to see this recommended in the Top10. I'm spending a lot more time talking with engineers about what challenges they're trying to solve, and why they architect their apps in different ways. 5 years back, I was talking to engineers about remediation timeframes for the 300th XSS finding in Fortify. This category helps recognise that change in relationship and culture between Security + Eng teams, while also representing that good security can be built in from the beginning. 

I hope that this being number 4 leads to more collaboration between sec and eng, but I've also seen a move to 'Checklist-driven-design' as an alternative. It's efficient to give a list of things to check (like the ASVS), but entirely impersonal and a bit of a dick move.  Anyway, helping engineers design automated solutions, squash bug classes, stay on the road, it's all good content. What do they say... Shift left? 

# 5. Security Misconfiguration

I like seeing the slight bump in priority as more resume-driven-development is adopted and engineers genuinely believe they'll be Facebook scale one day. The more complex your app is, the easier it is to make a misconfiguration. Since we're all doing development in AWS and using Kubernetes for our blogs now, i can totally see why this is rising in importance. Ironically, kubernetes and cloud services are allowing you to automatically and repeatably configure things. Sometimes they're just configured poorly across the entire fleet though so... not a silver bullet.

# 6. Vulnerable and Outdated Components

Vulnerable and Outdated Components is usually an automatic finding in every report I write. In my experience, it's a critical problem for legacy architectures where folk are reluctant to change or patch applications. The system might not even have any support anymore. I've had to help migrate components where the primary maintainer had passed away too :( 
    
On a more positive note, things are much better outside of the enterprise. Plenty of automated tools exist to scan packages now and pinning versions / automatic patching are standard practice. I like that the focus isn't necessarily on identifying vulnerable components in the provided guidance anymore. Rather, asset management, removing things you don't need, and proactively preparing for deprecation are key strategies mentioned within the category.

One thing I dislike about this category though is that Security Engineers tend to rely on tool outputs without contextualising the vulnerability to the app. It's certainly easier to throw 100 dependency problems over the fence to our engineer friends. But better outcomes stem from working with them to understand if the patch is entirely necessary, whether an alternative library is better to use, or if the function can be rewritten. Heck, deleting 99% of jQuery would remove the majority of vulns and still have a workable product haha. I wish the OWASP guidance included this too.

# 7. Identification and Authentication Failures

As I see more use of third-party authentication services, SSO/SAML tokens being passed around, OAuth2 stuff, Auth things, I'm not surprised this is here. I get that this is a push to get more engineers to include Multi-Factor Authentication in their apps. But it does make me a little sad that secret-question advice, brute forcing credentials, and session management are still top of the list. There has been lots of innovation and changed guidance over the last few years in the Authentication space. Killing password complexity, implementing blocklists for known bad passwords, FIDO2 and WebAuthN implementations. Plenty of options to bring developers into the new Authentication world.

# 8. Software and Data Integrity Failures

With javascript having SRI available for a few years (let alone apt-get), I didn't think it would take long until verification measures reached other parts of the ecosystem. While this is a new entry, making sure to verify the integrity of the file you receive has always been a consideration for security over the extent of my career. They specifically mention CI/CD systems, which I think is true enough. If you're not verifying the hash and signature of a file you receive, is it fine? I just think that DevOps engineers forgot how important this extra step is, and it's nice to see it popping back up again. 

# 9. Security Logging and Monitoring Failures

Don't have much to say for this except duh. Telemetry options are everywhere now with NewRelic, Airbrake, Airflow, Datadog, SumoLogic. Heck, go be a hunk of splunk. Or just all the native cloud stuff too. There's not really any excuse outside your enormous AWS bill from 7 year retention of kinesis events...

# 10. SSRF

I think it's a little weird to see since it's an injection problem. But like XSS before it, I think it's good to call out explicitly this time. These issues are popping up regularly in bug bounty and pentests, especially in the 2021 MS Arch world, so glad to see it getting formal recognition.

# Conclusions

The Top10 is an enormous project that takes a lot of time to pull data, do the math, and turn the results into something consumable by developers of all skill levels worldwide. I'm glad to see it being updated for our 2021 world and hope you're excited for the 20th Anniversary of OWASP in a few weeks time.

As always, feel free to write back to me on LinkedIn and til next time!

Cole Cornford