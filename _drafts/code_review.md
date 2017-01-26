---
layout: post
title: My top tips for code review
permalink: 2017-28-01-creating-blog-with-jekyll
---

TODO intro

1. Do small reviews often
10 lines of code = 10 issues. 500 lines of code = "looks fine" (via [@iamdeveloper](https://twitter.com/iamdevloper/status/397664295875805184?lang=en)) Not only we are lazy and we dont want to cope with huge code diffs but doing small reviews means fast feedback. With fast feedback you can avoid loosing a week work because you forget that one little thing at day one. What I do is to start a pull request with first commit. I will prefix such commit with WIP to indicate that it is not ready to be accepted. Gitlab has [nice support](https://docs.gitlab.com/ee/user/project/merge_requests/work_in_progress_merge_requests.html) for this. Everyone is invited to comment and stop me from doing mistakes in the very beginig. And if you want to do continuos integration you should merge at least once a day anyway.

ttt. [Look on the bright side of code](https://www.youtube.com/watch?v=SJUhlRoBL8M)
Do not comment only problems. Pick something you liked and mention it to. 

5435. No new work when there is a review
When there are PRs to review no new work is started. This is also essential for continuos integration. 

435. Juniors should review to
Fresh air into the code. When you do one job long enough you become blind to some problems. Young team member can disrupt this and push you into solving it with completely new approach or framework. 

666. Do two rounds technical and domain
Check not only if there is no null pointers but if the code does make sense from business point of view

54. Use tools
Static code analysis is your friend. [Codeacy](https://www.codacy.com/projects), [SonarQube](https://www.sonarqube.org/) and [many others](http://halyph.com/blog/2011/08/31/java-code-quality-tools-overview.html)

rrr. Ego aside
You are commenting the code not the person who wrote it. After all [you are not your code](https://blog.codinghorror.com/egoless-programming-you-are-not-your-job/). Same applies for the author. Dont see comments of your code as a harrasment but as a way to learn. Open your mind to other points of view.

yyy. Start with tests
You start implementaion with test, am I right? I apply same for code review. If there are no tests you can stop right there. Never accept code without tests. When tests are horribly written indicate that there are some problems with the code itself. Because what is not testable is usually not very usable.  

xxx. Not only code needs review
Review also commit structure and messages. Documentation, readmes, etc..  
 