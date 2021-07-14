---
layout: post
title: How to Backup Your Data to Azure Using Duplicity
date: 2021-07-13 00:00:00 +0100
description: Use cheap Azure Storage to backup your data on a regular basis using Duplicity
tags: [Backup, Duplicity, Azure] # add tag
toc: false
comments: false
image:
  src: /assets/img/jorgen-haland.jpg
  alt: Sheeps
---

# Why Azure Storage and Duplicity?
Azure Storage Accounts are a great place to store your backup files. It's cheap, as cheap as 10 dolars/month for 1 Terabyte of storage (based on cool tier) plus what you pay to read data out of the Storage Account. Though it may be cheaper to just buy Onedrive storage and use it, but I haven't tried that option. 
Duplicity encrypts all the data before sending it to Azure and it has incremental backups which effectively saves some space.
Ultimately I decided to go this route because I have an Azure account ready to go and want to use it more.

# How to Backup Your Data to Azure Using Duplicity

Automated encrypted backup of your data to an Azure Storage Account.

We'll be using `duplicity` to perform the backup: <http://duplicity.nongnu.org/>
# Steps

### Prep work

Create a file that will store the passphrase, azure storage key and azure storage account name

```bash
AZURE_ACCOUNT_NAME="yourstorageaccountname"
AZURE_ACCOUNT_KEY="storageaccountkey"
PASSPHRASE="banana123"
```

Save the file under `/root` and only allow root itself to read it

```bash
cat > /root/.passphrase << EOF
AZURE_ACCOUNT_NAME="yourstorageaccountname"
AZURE_ACCOUNT_KEY="storageaccountkey"
PASSPHRASE="banana123"
EOF

chmod 700 /root/.passphrase
```

### Azure backend

If you want to use azure as backend you need to have `python3` installed and the Microsoft Azure Storage SDK for Python ([https://pypi.python.org/pypi/azure-storage/](https://pypi.python.org/pypi/azure-storage/)) version 0.36.0 or earlier.

### Create cron backup script

We will create a backup script with the following content:

```bash
#!/bin/sh

test -x /usr/bin/duplicity || exit 0
. /root/.passphrase

export PASSPHRASE
export AZURE_ACCOUNT_NAME
export AZURE_ACCOUNT_KEY
/usr/bin/duplicity <source> azure://<container_name> # no subfolders are allowed
unset PASSPHRASE
unset AZURE_ACCOUNT_NAME
unset AZURE_ACCOUNT_KEY
```

I want to set duplicity to create daily incremental backups.

```bash
cd /etc/cron.daily
cat > duplicity.sh << EOF
#!/bin/sh

test -x /usr/bin/duplicity || exit 0
. /root/.passphrase

export PASSPHRASE
export AZURE_ACCOUNT_NAME
export AZURE_ACCOUNT_KEY
/usr/bin/duplicity <source_folder> azure://container_name # no subfolders are allowed
unset PASSPHRASE
unset AZURE_ACCOUNT_NAME
unset AZURE_ACCOUNT_KEY
EOF

```

### Restoring data

To restore the data is quite straighforward and is basically just reverting the parameters passed to `duplicity`.
Basically:
```bash
/usr/bin/duplicity azure://container_name <destination_folder>
```

A full restore script would look something like:
```bash
#!/bin/sh

test -x /usr/bin/duplicity || exit 0
. /root/.passphrase

export PASSPHRASE
export AZURE_ACCOUNT_NAME
export AZURE_ACCOUNT_KEY
/usr/bin/duplicity azure://<container_name> <destination>
unset PASSPHRASE
unset AZURE_ACCOUNT_NAME
unset AZURE_ACCOUNT_KEY
```