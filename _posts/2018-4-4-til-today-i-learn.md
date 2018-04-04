---
layout: post
title: Today I learn
subtitle: A note about what I learn a day, it can be anything, not only iOS. 
date:   2018-4-4
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftoday-i-learn%2Fnever-stop-learning.jpg?alt=media&token=018d6c76-c64a-42a6-9b6c-317ec9b8b1d6'
---

# 4/4/2018

### How to remove pod in iOS project
```
sudo gem install cocoapods-deintegrate cocoapods-clean
pod deintegrate
pod clean
rm Podfile
```

### Integrate Zendesk SDK 

Do not follow pod instruction from Zendesk tutorial, take too much time to fix but can't run. 

Add SDK manual way: Add 2 more framework from iOS `MobileCoreServices.framework` and `Security.framework.`

Access to Tagets/General, add all Zendesk SDK files to Embedded Binaries.

### Merge unrelated histories in git 
Usually have this issue when create new project in local, create repo in git then add remote url from git to local project.

`git pull origin branch_name --allow-unrelated-histories`