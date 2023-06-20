## Commits
Show all the commits:
```bash
git log -p
commit edc5aabf933f6bb161ceca6cf7d0d2160ce333ec (HEAD -> master)
Author: SherlockSec <dan@lights.htb>
Date:   Fri May 31 14:16:43 2019 +0100

    Added some whitespace for readability!

diff --git a/bot.js b/bot.js
index e582ba9..f04294b 100644
```
With the hash of the commit or part of it we can go to such commit:
```bash
git checkout -b new_branch 3359900
```