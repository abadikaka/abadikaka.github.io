---
# Common-Defined params
title: "Trigger iOS Unit Test Check-In Github with Jenkins (Automation)"
date: "2020-07-31"
description: "Learn to deploy unit-test check-in with Jenkins"
categories:
  - "Automation"
tags:
  - "CI/CD"
lead: "Learn to deploy unit-test check-in with Jenkins" # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
pager: true # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: "right" # Enable sidebar (on the right side) per page
widgets: # Enable sidebar widgets in given order per page
  - "search"
  - "recent"
  - "taglist"
  - "social"
draft: false
---

Wonder how to make automation for triggering the build and unit test in git with Jenkins? So this one could be the easiest tutorial you can follow. First let open our Jenkins web page, login, and create a new Free project. Once you create it, go to configure tab and you will go to this page below

![jenkins](/img/test-jenkins-1.png)

Fill up the textbox above on the description and tick the GitHub project row and fill in the project URL. Scroll down until you see the Restrict where this project can be run, and tick it if you have a specific node being set up on your machine. If not, you can ignore this option.

![jenkins](/img/test-jenkins-2.png)

Now go over to Source Code Management tab. Fill the blank column for Repositories row with your data (Fill it depend on your configuration). After that in Branches to build row, fill in `${ghprbActualCommit}`. What does it mean? Those are the environment variables for GitHub pull request builder. Check this [Github PR Builder](https://plugins.jenkins.io/ghprb/) link for more information.

![jenkins](/img/test-jenkins-3.png)

Go to **Build triggers** tab. Fill and tick the **GitHub PR Builder** and add the correct credentials and admin list on the Jenkins. For the **Skip build phrase** and **Crontab line** is optional, you can fill it the same or leave it with the default one.

![jenkins](/img/test-jenkins-4.png)

Now go to this section and don’t forget to tick the Build every PR Automatically without asking (Dangerous!). This one is so important, tick it to make it trigger the automation. Also, you can put either blacklist or whitelist target branches for triggering the unit test check.

![jenkins](/img/test-jenkins-5.png)

Go to the fun part! Let build the unit test checker. Fill in the commit status context in the **Trigger Setup** row and the build result for **SUCCESS** and the message itself. (You can set the **FAILURE** message as well).

![jenkins](/img/test-jenkins-6.png)

Go down to this **Build Environment** section and also tick this row and put the same thing with above. If you put differently then you will trigger 2 different checkers. Actually you can ignore the first one if you use the **GHPRB** Trigger in the beginning.

![jenkins](/img/test-jenkins-7.png)

For the last tab go to **Advanced Xcode build options** and put the **Custom xcodebuild arguments**. Fill the scheme the same as your product scheme. You can add more custom arguments base on your necessary requirements.

![jenkins](/img/test-jenkins-8.png)

Move over to your GitHub repo, once you trigger a new commit, it will try to trigger Jenkins to build and run the **xcodebuild** and give us the feedback whether its success or not.

![jenkins](/img/test-jenkins-9.png)

Finally, you are able to make your first setup in Jenkins!. Let me know what do you think
