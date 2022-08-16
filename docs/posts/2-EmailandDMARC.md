---
layout: default
title: Emails and DMARC
nav_order: 2
parent: Posts
---

# Issue: Not receiving Salesforce emails

**Jump to**: [Scenario](#scenario) / [Investigation](#investigation) / [Background](#background) / [Solution](#solution)

### Scenario:
I set up a new org and added users. While I was doing some testing, I noticed I was not getting certain Salesforce emails (ex: if a Case was assigned to me). Also, when I was testing a Flow that had an action to send an email to a Queue, the Flow was also failing. From the failed Flow interview, I received the error “0 recipients”. I did also receive an email from Salesforce with the same error information, so some emails were coming through.

### Investigation:
I requested the logs by going to Email Logs within Setup. This provided the exact information I needed to realize what the problem was.  

<span style="font-size:9px">(Log has been edited and unnecessary columns hidden) </span>
![email log](/assets/images/2/2_email-log.png)  

Here I saw the issue was related to my domain's DMARC policy. Salesforce used the current user's email address when sending the email, so in my case the email that was sent to the Queue within my Flow was going to be sent with my own user's email as the sender address. Normally businesses will opt to use an Organization-Wide Email Address as the sender email for Flows and other automations. For example, you might want to use “helpdesk@orgname.com” or “info@orgname.com” as the standard for all your email communications. If I had set up an Org-Wide Email and added it to the appropriate setting within Process Automation Settings, it wouldn't have using my user's email address as the sender. However, I did not have that set up for this particular Org. But even if I had, in that case I would still have run into this same issue, because I would have set the Org-Wide Email to another email within the same domain that had the DMARC policy.

### Background:
DMARC is a policy that states what to do with emails that fail SPF and DKIM checks, both of which are a way for domains to prevent unauthorized email from their domain. If you're unfamiliar with those terms, I won't go into detail here about what each of them means. I'll just explain in general terms in order to provide some context as to why I encountered the issue.  

Let's use an analogy and think of an email like a physical letter that you put in an envelope and send via the postal service. I could write your name and address as the “from” information on an envelope and send the letter from some random location, pretending to be you. The postal service doesn't have any way to check whether or not that letter actually came from you so they would just send it anyway. Email is like that too. For example, I could spoof an email address “admin@amazon .com” and use it to send email to other employees at Amazon, with the intention of tricking them into opening the email because it appeared to be a legitimate account from their own organization. One thing companies can do to prevent this spoofing is to use DMARC, DKIM, and SPF. Again, I won't go into definitions or technical specifics here, but I may do so in another post.  

In my case, Salesforce was essentially pretending to be my domain by putting my address as the sender but sending it out from their mail hosts. I enforce both DKIM and SPF on my domain to prevent this exact thing from happening. In this case I actually wanted Salesforce to send these emails, but normally this is set to prevent outside spammers from sending email as me. So DKIM and SPF failed, and the DMARC policy specified that emails like that are rejected. But the error message with why the email was rejected gets recorded, which is what I saw in the Email Logs.

### Solution:
In my case, it ended up that SPF didn't fail, I think because Salesforce uses the same email provider that I have specified as authorized senders on my SPF record. If I had a different provider with different mail hosts, or if I had my own on-premises mail server, SPF in those cases would fail. So then if SPF had failed, I would have needed to update my domain's SPF record with Salesforce's email hosts to authorize them to send email on behalf of me.  

DKIM did fail for me, so I had to put a solution in place to prevent that. Salesforce has a feature in Setup called DKIM Key. Once I created a key, I also added a CNAME record on my domain that included this information. Once the changes propagated, in Salesforce I was then able to click the Activate button for the DKIM key.  

![email log](/assets/images/2/2_dkim-settings.png)  

And that was it! Now that I had given Salesforce this authorization to send email on behalf of me for my domain, I was no longer getting the same error during my Flow testing, and all future emails were received.  

The originating problem may not always be encountered, especially if you're working with an Org that was already set up before you got there. It was a lot of fun for me to troubleshoot this and I'm glad that my history as an IT Systems Administrator came in handy, because I already had worked with and understood DMARC, DKIM, and SPF.