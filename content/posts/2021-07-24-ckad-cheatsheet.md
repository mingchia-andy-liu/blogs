---
title: "CKAD cheat sheet"
date: 2021-07-24T17:41:22-07:00
description: This post tells you how to ace CKAD and give you my cheat sheet!
images:
  - images/ckad-certified-kubernetes-application-developer.png
slug: ckad-cheat-sheet
tags:
  - kubernetes
  - ckad
type: post
---

{{< figure
    src="/images/ckad-certified-kubernetes-application-developer.png"
    alt="CKAD badge."
    position="center"
    width="50%"
>}}

## Introduction

I recently passed the Certified Kubernetes Application Development exam on July 2021.

Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications. [The Linux Foundation](https://www.linuxfoundation.org/) offers two certifications on this technology: Certified Kubernetes Application Developer (CKAD) and Certified Kubernetes Administrator (CKA). I wanted to share tips that are helpful while preparing the exam and during the exam as well.


## Exercises

The most important thing is to practice. The exam is about live and breath `kubectl` commands. If you need to write out the yaml files manually, you will not have enough time to complete the exam.

[killer.sh](https://killer.sh) is a great place to practice. You have 2 free sessions with the exam purchase. The questions are harder than the actual exam so it is a great way to see if you are prepared or not.

Once you are done with the `killer.sh`'s sessions. You still have access to the cluster 36 hours after you've started. You can use time to practice with other questions. I used up all 36 hours to practice questions here over 3 times. After that, I felt comfortable going to the exams.

This is a good collection: https://github.com/dgkanatsios/CKAD-exercises.

### Internet connection

Test your internet connection by setting up a zoom/meet call with yourself and turn on your camera. This will simulate the exam bandwidth and allow you to see if you have good enough internet. I would suggest do this in parallel of the killer.sh's practice exam. It can tell you if your internet can stream and connect to a remote cluster in a timely manner.

## Practical tips

### Terminal set up

These three simple steps helped me the most in terms of saving times. It was much faster to look up something without typing the same commands over and over again
```
alias k=kubectl
alias kn='kubectl config set-context --current --namespace '
​
export do="--dry-run=client -o yaml" # like short for dry output. use whatever you like
```

#### vim

To make vim use 2 spaces for a tab edit `~/.vimrc` to contain:
```
set sw=2 ts=2 sts=2
set expandtab
```

If you need to paste something from another file, ie. pod spec, use `set paste`. This will format correctly. Then you can turn on visual mode, `shift+v` and use `>` or `<` key to indent the lines.

## Exam

#### Correct cluster and namespace
4 clusters are provisioned for your exam, and each question provides the command to switch to the correct cluster. Make sure you copy paste and run that command before each question.

Always switch back to `default` namespace if the question does not specify one.

These are the 2 commands after I start a new question. `kn default` and the command to connect to the correct cluster.

#### sudo

I did not have to use `sudo` during the exam despite the many people have claimed that the fs is blocked with provisioned user.

#### Delete pods

If you make a mistake and want to delete a pod, use `--force --grace-period=0` flags. Deleting a pod takes a long time and you want to save much time as possible. Use these flag will exit the delete command immediately.


---

Trust yourself. You have practiced enough and don't panic. You have a free retry if you fail the first time.

Good luck 👍
