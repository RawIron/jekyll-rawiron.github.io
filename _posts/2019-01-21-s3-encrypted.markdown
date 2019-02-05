---
layout: post
title:  "Store client-side encrypted data in the cloud using rclone and encfs"
date:   2019-01-21 14:39:12 +0200
categories: devops s3
---

Everyone has data she would not like to share.
And everyone has data she would not like to loose.

Before the data is transfered to a cloud storage solution it is encrypted on the client.
The clear text of the data is only known to the client.

The cloud storage provider cannot read it and the data is safe.

The tools to build this:

* awscli
* encfs
* [rclone][rclone-home]


## Setup

### S3

{% highlight bash %}
sudo apt install awscli jq
{% endhighlight %}

[Here][s3-create-bucket] is a nice step-by-step instruction for creating a bucket from the command-line.

* create a S3 bucket
* create a role with read and write access to this bucket
* generate the access key pair for this role

Save the iam policy to a file named _iam-s3-storage.json_
{% highlight bash %}
{% raw %}
cat <<IAM_POLICY >iam-s3-storage.json 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:DeleteObject",
        "s3:GetObject",
        "s3:PutObject",
        "s3:PutObjectAcl"],
      "Resource": [
        "arn:aws:s3:::private-data-storage-9e9ace/*",
        "arn:aws:s3:::private-data-storage-9e9ace"
      ]
    }
  ]
}
IAM_POLICY
{% endraw %}
{% endhighlight %}

{% highlight bash %}
# use key pair with rights to
#  create a bucket in S3
#  create user, policy and access key pair in IAM
aws configure

# make the bucket name pretty and unique
aws s3api create-bucket --bucket private-data-storage-9e9ace --region ca-central-1 --acl private --create-bucket-configuration LocationConstraint=ca-central-1

# manage access to the bucket
# create a user
aws iam create-user --user-name s3-storage

# policy with read, write, list privileges
aws_s3_policy_arn=$(aws iam create-policy --policy-name s3-storage-rw --policy-document file://iam-s3-storage.json | jq '.Policy.Arn' | sed 's/"//g')

# give the user access
aws iam attach-user-policy --user-name s3-storage --policy-arn ${aws_s3_policy_arn}
# alternatively give a group access
# and assign user to the group

# create and get the access key pair
aws iam create-access-key --user-name s3-storage
{% endhighlight %}

### rclone

{% highlight bash %}
sudo apt install rclone
{% endhighlight %}

Create a config file like the one below in _~/.config/rclone/rclone.conf_.

{% highlight bash %}
{% raw %}
mkdir -p ~/.config/rclone
cat <<RCLONE_CONF >>~/.config/rclone/rclone.conf
[s3-ca]
type = s3
env_auth = false
access_key_id = <replace-me-with-your-access-key>
secret_access_key = <replace-me-with-your-secret-key>
region = ca-central-1
endpoint = 
location_constraint = ca-central-1
acl = private
server_side_encryption = 
storage_class = 
RCLONE_CONF
{% endraw %}
{% endhighlight %}

### encfs

{% highlight bash %}
sudo apt install encfs
{% endhighlight %}

[This][encfs-step-by-step] is a short and good how-to for _encfs_.
A long key (password) for the encryption is a good idea.
Any password safe is the best place to keep the key.

{% highlight bash %}
mkdir ~/cloud-encrypted
mkdir ~/cloud-storage

encfs ~/cloud-encrypted ~/cloud-storage
{% endhighlight %}

> Do not delete or lose the .encfs.xml

The mount is ready to use.

{% highlight bash %}
findmnt
{% endhighlight %}

## Encrypt, Decrypt and Access the data

The encrypted data can be viewed in clear-text when the _encfs_ filesystem is mounted.
To hide the data simply unmount the _encfs_ filesystem.

{% highlight bash %}
fusermount -u ~/cloud-storage
{% endhighlight %}

When the encrypted data is needed, mount _encfs_.

{% highlight bash %}
encfs ~/cloud-encrypted ~/cloud-storage
{% endhighlight %}

Manage the data as usual in the _~/cloud-storage_ directory.


## Sync the data to and from S3

_rclone_ is the tool to transfer the data between the client and the cloud storage.

### Transfer Data

Copy the encrypted data to the cloud.

{% highlight bash %}
rclone copy ~/cloud-encrypted s3-ca:private-data-storage-9e9ace
{% endhighlight %}

Restore the encrypted data from the cloud.

{% highlight bash %}
rclone copy s3-ca:private-data-storage-9e9ace ~/cloud-encrypted
{% endhighlight %}


[s3-create-bucket]: http://notes.webutvikling.org/add-s3-bucket-using-awscli-example/
[rclone-home]: https://rclone.org/
[encfs-step-by-step]: https://www.howtogeek.com/121737/how-to-encrypt-cloud-storage-on-linux-and-windows-with-encfs/
