# Pwned Labs Cloud Security Labs

What is up dickwad!

So this is the broke option for you: go to the Pwned Labs website and create yourself a free account. They have four cloud labs that you can do, that will teach you what a cloud security engineer does, like going through log files, identifying suspicious users, following their trail, and what exactly makes them suspicious.

Two labs for AWS and two for Azure, do whichever you choose. Once you follow their instructions to find what they eventually did, you traverse the same path as them to create a proof of concept for your final report.

---

## Lab 1

My lab 1 where I got the flag:

Basically we get logs from a company that we then traverse and pinpoint different users, identify a suspicious one, then follow their actions to the point where they exfiltrated data. Then we act like the attacker and follow the same path as them, using the company's AWS IAM account with the same initial privileges, and attempt the data exfiltration to prove the vulnerability.

![Lab 1 - step 1](/images/Screenshot2026-09-09192737.png)
![Lab 1 - step 2](/images/Screenshot2026-09-09192801.png)
![Lab 1 - step 3](/images/Screenshot2026-09-09192808.png)
![Lab 1 - step 4](/images/Screenshot2026-09-09192827.png)

---

## Lab 2 (This is Amazon's service Macie)

It's teaching you what the service is (a data privacy and protection service), and how to enable and use it via the web interface (console) and CLI. DON'T BE A BITCH, DO BOTH.

Please don't try remembering the commands in the CLI, just understanding should be fine, at least that's what I did.

**Summary:** Amazon Macie is a paid service, the payment depends on your region and the amount of data you have, and could cost a few dollars or a few thousand.

It lists all your S3 buckets, finds sensitive data, shows how it can be accessed, and then you can fix that and secure it. What data it looks or doesn't look for is something you can change. (I did have an idea that if I'm someone responsible for setting up a cloud environment, after the final touches I deploy it first in a region in which Amazon Macie and other services cost the least, run it, fix any issues, and then redeploy in the preferred region.)

---

## Lab 3: Identify IAM Breaches with CloudTrail and Athena

**Summary:** This is where you are going to be learning another Amazon service called Athena. It's basically a powerful and fast way to query the CloudTrail logs and get results quick, a useful tool, and the only one I have discovered so far that I would rather use in the console as opposed to the CLI. In the lab we are trying to identify a breached IAM account.
