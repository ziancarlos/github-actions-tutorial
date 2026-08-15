(base) zian@Zians-MacBook-Pro Github Actions % >....                                             
normally output is

  feature-restructure
* master

the star meaning that are the branch we are working in

3. Delete Branch
git branch -D <<feature_name>>

4. After creating a new branch and commit we can try git log
commit bdda025f83a392634c1ccb7cb935d56ee4735e86 (HEAD -> feature-restructure)
Author: Zian Carlos Wong <ziancrlswong@gmail.com>
Date:   Sat Aug 15 16:30:01 2026 +0700

    add restructure page

commit a221d59fb8a6a9c9abfb1357eca5e21192828b0f (master)
Author: Zian Carlos Wong <ziancrlswong@gmail.com>
Date:   Sat Aug 15 14:31:13 2026 +0700

    initial commit

commit 40dd19fae301a9cbdc92e329f4f0bda4e39583fc
Author: Zian Carlos Wong <ziancrlswong@gmail.com>
Date:   Sat Aug 15 14:30:41 2026 +0700

    initial commit

this will show the master/main branch is behind our new branch/feature branch


5. merge a branch into current branch
example iam in master/main branch i want to merge with feature-restructure branch
1. git checkout master/main -> if not
2. git merge <<branch_name>> 
3.