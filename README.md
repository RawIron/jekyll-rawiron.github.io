# My Jekyll Blog

A more complicated way to maintain the blog on *github pages* was needed because *github pages* does not support all *jekyll plugins*.


## Add a New Blog Entry

* create a new branch for post
* create a file under *_posts*
* build the blog
* check the new post on the local machine
* once the blog post is done merge the branch into master
* copy the new version of the blog to the *rawiron.github.io* repo


```bash
# each post on a different branch
gco -b issue-12-spark-jupyter-notebook

# write the post ...
sublime _posts/2017-07-02-setup-jupyter-notebook-for-spark.markdodwn

# build the blog
bundle exec jekyll serve

git rebase master
git push origin
# merge the branch on github

# deploy the blog
cd ../rawiron.github.io
cp -r ../jekyll-rawiron.github.io/_site/* .
ga _site/
git commit "add notebook for spark post"
gp origin
```
