# GIT

# git init

> can change a normal directory into a git repository.

1. cd into this directory

2. use ```git init``` command in terminal

3. Then a .git file will be create, which means this directory becomes to a git repository

   *if a project download from Github as .zip and you unarcheve this .zip, at this time, it is only a normal directory, and you need to use ```git init``` command to change it to a git repo.*



# git clone HTTPS-links

> clone a repository from somewhere

In contrast to ```git init``` , use this command to download can create a git repository automatically.



# Commit

Look how to do it in the [Github Docs](https://docs.github.com/en/repositories/working-with-files/managing-files/adding-a-file-to-a-repository#adding-a-file-to-a-repository-using-the-command-line).

```
git add .
git commit -m "Add existing file"
git push origin YOUR_BRANCH
```

**fine, I believe that using VScode as a tools to control git is far more convenient than command line.**

and I have a basic question. That is, at the frist time I tried to commit&push my changes to Github, it works, but at the place shown who contribute to this commit, that name and head picture is not mine. I dont know how to change it to my own so i delete this reository and create a new one, then it works well..  Now i still dont know how and why about this question, maybe i need to ask AI to seek for help..