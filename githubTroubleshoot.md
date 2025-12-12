## How does github authenticates using ssh key?
You generate a ssh key
```bash
ssh-keygen -t ed25519 -C "email@example.com" 
```
you copy the public key to the ssh keys in your github account

You have the private key in your `~\.ssh\id_ed25519` which is used by github

but here is the thing

You configured remote as origin with `git@github.com:revanthreddydatla/TheOperationsGuy.git` here

**username** is git  
**host** is github.com
**repo address** is revanthreddydatla/TheOperationsGuy.git

git doesnot use revanthreddydatla to find account.

git client actually sends the signature of private key and github uses that private key to find
the owner of associated public key of that.

So if you have used same public key for two github accounts, then the first account where the public key is stored will be treated as the user pushing to the repo.

Example:
github_userAccount1
guthub_userAccount2

you generated a keypair and copied public key to both github accounts (Note: Actually github doesnot allow adding same key), 
as github uses private key to find owner of public key, here it causes unexpected results.