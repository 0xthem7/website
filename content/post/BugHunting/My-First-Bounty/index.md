---
title : My first Bug Bounty reward  
description : This is How I earned my first 100$ via bug bounty hunting 
slug : my-first-bug-bounty-reward
date : 2024-10-2 00:00:00+0000
image : cover.png
categories :
    - Bug Hunting
tags :
    - writeup
weight : 1
---

## Background

I've been looking around the **BugCrowd VRT** as well as keeping eyes on certain programs. Genreally I hunt for a bit and then again update my knowledge via blogs or any other medium and the cycle continues.

While doing so I decide to Approach a program on BugCrowd, I dedicated some hours and started poking around the application. 

Mostly I hunt the main program in the beginning and then, I start recon for the subdomain enumeration and all.  

Starting with the main program after spending quite a lot of time on **login** feature with zero sucess,  
I moved to file upload and deleting the file. 

As I moved forward I captured an interesting endpoint which was used to delete the post of the users.

## Capture and Analyze

I captured the request of the program and it was hitting like the following endpoint 

```****/accounts/media/delete/{content type}/{post_id}```

***content type** - photo, video, gif*

***post_id** - You can find the post id if you open the post  
            lets say image name is something-cool-998123 the number of the post is post_id which is in this case 998123*

This endpoint is deleting the post of the user via over GET request
## Validating the issue


I have configured Pwnfox for managing containers over my firefox browser and created two accounts let's Attacker and Victim.


Victim - Created an post.  
Attacker - Crafted a url and sent message to the Victim something like ```****/accounts/media/delete/photo/998123```  
Victim - Clicks the url and the post get's deleted  

## Submission and faliure

I submitted the bug and soon after I got informed that...  
**This vulnerability has already been reported by another researcher and marked it as duplicate**

I was like okey lets move on and got busy on some other task 

After few days I received an email that this issue has been **resolved**.

## A New dawn 

On the same program after few days...

I started normal recon and gathered multiple subdomain and then captured screenshots of the subdomains.


After visually inspecting  the subdomains,  
An subdomain caught my eye. It was the same replica of the main domain and it was perfectly sycing all the posts and updates as well as features of the main domain.


I tested the same vulnerability over this subdomain and it got replicated even though they have fixed the issue over the main domain  
But it was not fixed on the subdomain.

I reported the vulnerability and after someday it got triaged and I was rewarded with 100$.

## Thoughts 

Bug hunting is like finding the **One piece** , Someone maybe the **Gold D. Roger**. But you could be the Next **Monkey D. Luffy**

**One Piece** - Specific Bug on a specific program over a specific endpoint   
**Gold D. Roger** - Duplicate Bugs   
**Monkey D. Luffy** - New generation / Looking for New bugs  

Obviously there are multiple **One Piece** in the world of Cybersecurity you just have to keep for looking it and improving yourself accordingly.
