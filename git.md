问题：
```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
解决：
```
ssh-add --apple-use-keychain ~/.ssh/github
```