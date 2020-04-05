---
templateKey: blog-post
title: Transferring an app to another Firebase account
date: 2020-04-05T04:57:47.564Z
description: >-
  This page describes how to use the managed import and export features to move
  Cloud Firestore data from one project to another. 
featuredpost: true
featuredimage: /img/1_3kPOI1_HGuE0fPWBj_jnog.png
tags:
  - Google Cloud
  - Firebase
  - Technology
---
###### To transfer firestore data login to google cloud account first and follow the steps below.

#### Create a Cloud Storage bucket to hold the data from your source project.

![Navigate to Storage and select Browser.](/img/browser.PNG "Navigate to Storage and select Browser.")

**Step 1:** Navigate to Storage and select Browser.

**Step 2:** Create New bucket.

**Step 3:** Begin an export operation

* Steps to export all data from source bucket. (In cloud terminal)

```apex
1. gcloud config set project [SOURCE_PROJECT_ID]
2. gcloud firestore export gs://[SOURCE_BUCKET] --async
```

* Take a note of outputPrefix

  `gs://[SOURCE_PROJECT_ID]/2020-03-31T18:06:31_1640`
* Go to **\[DESTINATION_BUCKET]** --> Permissions, existing member or new member under Storage select **Storage Object Creator** Role.

**Step 4:** Begin an import operation

* Steps to import all data to destination bucket. (In cloud terminal)

  ```
  1. gcloud config set project [DESTINATION_PROJECT_ID]
  2. gcloud firestore import gs://[SOURCE_BUCKET]/[EXPORT_PREFIX] --async
  ```

#### **Transfer Firebase Authentication data to different account.**

<!--StartFragment-->

![Firebase Android Series : Authentication - ProAndroidDev](https://miro.medium.com/max/1024/1*zTdZMxbTkVdXCOoZlXLnsg.png)

<!--EndFragment-->

**Step 1:** Begin an export operation

* Install The Firebase Command Line Interface (CLI) Tools in your system.

    `npm install -g firebase-tools`
* Authenticate to your Firebase account

  `firebase login`
* Export all authentication data

  `firebase auth:export AllUsers.json --project projectId`

**Step 2:** Begin an import operation

* Before import command specify the hash key. Hash key is different for all the accounts.

  Hash key can be used to migrate users password.

  ```
  hash_config {
    algorithm: SCRYPT,
    base64_signer_key: <long string of random characters>,
    base64_salt_separator: <short string of random characters>,
    rounds: 8,
    mem_cost: 14,
  }
  ```
* Import command 

  ```
  firebase auth:import ./AllUsers.json --hash-algo=scrypt --rounds=8 --mem-cost=14 --hash-key=<long string of random characters> --salt-separator=<short string of random characters>
  ```



#### Transfer Firebase Storage to different Account

![Firebase   Storage](/img/storage.png "Firebase Storage")



###### You can create a transfer job in the GCP Console. You can specify source/destination buckets from different projects as long as you have access permissions

```
gsutil cp -r gs://[PROJECT_A_ID].appspot.com/my_directory gs://[PROJECT_B_ID].appspot.com
```
