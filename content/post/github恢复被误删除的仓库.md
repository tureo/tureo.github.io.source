---
title: "Github恢复被误删除的仓库"
date: 2021-10-26T15:32:40+08:00
lastmod: 2021-10-26T15:32:40+08:00
draft: false
keywords: []
description: ""
tags: [git,github,删库,恢复]
categories: [github]
author: "tureo"
---
有时候不小心误删除github仓库，github官方提供一种恢复的方法，这里以恢复误删除的file仓库示例，操作如下：
1. 登录GitHub，点击设置
<br>
![](/images/restore-deleted-repo/restore-deleted-repo-1.png)

2. 点击仓库，已删除的仓库，恢复（注意这里的提示，删除的仓库可能需要一个小时左右才能看到，刚删除的仓库这个页面可能没显示出来，需要等一会；还有不能恢复fork的仓库）
<br>
![](/images/restore-deleted-repo/restore-deleted-repo-2.png)

3. 确认恢复
<br>
![](/images/restore-deleted-repo/restore-deleted-repo-3.png)

4. 等待恢复队列完成
<br>
![](/images/restore-deleted-repo/restore-deleted-repo-4.png)

5. 完成恢复
<br>
![](/images/restore-deleted-repo/restore-deleted-repo-5.png)

6. 已删除的仓库列表已没有file仓库了
<br>
![](/images/restore-deleted-repo/restore-deleted-repo-6.png)

7. 回到自己的仓库列表，file仓库已恢复
<br>
![](/images/restore-deleted-repo/restore-deleted-repo-7.png)


[参考官方文档](https://docs.github.com/cn/repositories/creating-and-managing-repositories/restoring-a-deleted-repository)

