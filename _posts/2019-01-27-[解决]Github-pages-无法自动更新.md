---
layout: post
title:  "[解决]Github pages 无法自动更新"
date:   2019-01-27 22:56
categories: github-pages 不作恶
---

最近出现了一个非常奇怪的问题：github pages无法更新，明明成功push到github的仓库中。

最近的一次成功更新时间为2019.1.12，等1.22日再次上传时，发现更新不了了。尝试过很多种解决方法，都不行，甚至将版本回退到1.12也不行，最后在论坛上找到了解决办法。

以下是对此问题的分析：

 1. 本地编译验证

    正常情况下，如果提交内容有问题无法通过检查，github会来一封错误邮件。但我的提交未触发错误邮件，而且在本地构建是能够编译成功并正常访问的，说明提交的内容并无问题

 2. 查看github的提交及活动记录

    Github pages的仓库上，在***commit*** 和***environment***里，可查看到相关记录，见下图

    ![menu]({{ site.url }}/assets/posts/[解决]Github-pages-无法自动更新/menu.PNG)

    ***commits***中每次提交都能查看检查结果，见下图

    ![checkresult]({{ site.url }}/assets/posts/[解决]Github-pages-无法自动更新/checkresult.PNG)

    ***environment***中也可以查看每次成功的活动记录，见下图

    ![activitylog]({{ site.url }}/assets/posts/[解决]Github-pages-无法自动更新/activitylog.PNG)

    但是非常奇怪的是，1.22号及之后的所有提交都没有检查及活动记录，也就是说并没有触发github pages的自动构建。

 3. 更新index.html

    怀疑index.html没改动，导致未触发github pages的自动构建，尝试更新index.html，还是不行。

 4. 上论坛找原因

    实在找不到原因的情况下，最好是上官方论坛逛逛，因为如果自查无问题，那么多半有其他人遇到过类似情况，→[Github官方论坛](https://github.community)，如果在论坛里也找不到原因，还可以联系github官方寻求解决办法。

    果然找到类似问题：[page doesn't update ](https://github.community/t5/GitHub-Pages/GitHub-page-doesn-t-update/td-p/17618)

    这个回答拯救了我

    ![ReGithubpagedoesnotupdate]({{ site.url }}/assets/posts/[解决]Github-pages-无法自动更新/ReGithubpagedoesnotupdate.PNG)



此问题原因在于：最近github官方对博客服务做了调整，不支持免费私有仓库上建的博客。

**两种解决方案：升级或转为公有仓库。**