---
title: "How to pass CKAD?"
date: 2021-07-24T17:41:22-07:00
description: This post tells you how to ace CKAD and give you my cheat sheet!
image: images/ckad-certified-kubernetes-application-developer.png
slug: ckad-cheat-sheet
tags:
  - kubernetes
  - ckad
  - certificate
---

{{< figure
    src="/images/ckad-certified-kubernetes-application-developer.png"
    alt="A CKAD badge"
    position="center"
    width="50%"
>}}

## Introduction

I recently passed the Certified Kubernetes Application Development exam in July 2021.

Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications. [The Linux Foundation](https://www.linuxfoundation.org/) offers two certifications on this technology: Certified Kubernetes Application Developer (CKAD) and Certified Kubernetes Administrator (CKA). I wanted to share tips that are helpful while preparing for the exam and during the exam as well.

&nbsp;

## Exercises

The most important thing is to practice. The exam is about live and breath `kubectl` commands. If you need to write out the yaml files manually, you will not have enough time to complete the exam. Just watching the online videos or completing a course will not guarantee you success in this exam. Practice makes it perfect.

[killer.sh](https://killer.sh) is a great place to practice. You have 2 free sessions with the exam purchase. The questions are harder than the actual exam so it is a great way to see if you are prepared or not.

Once you are done with the `killer.sh`'s sessions. You still have access to the cluster 36 hours after you've started. You can use time to practice with other questions. I used up all 36 hours to practice questions here over 3 times. After that, I felt comfortable going to the exams.

This is a good collection: https://github.com/dgkanatsios/CKAD-exercises. Another very useful source is practice problems given under https://www.katacoda.com/liptanbiswas/courses/ckad-practice-challenges. If you have to refer to the official document more than 2 times during your practice (excluding copying the yaml), I would say you are not ready for the exam.


### GCP/Azure and network plugins

If you are using a cloud providers to practice, make sure you have the latest version. The default version on GCP was behind the latest.

For Network Policies, make sure you have enable/setup a network plugin. It is one of the exam topics but the feature depends on the underlying network plugin.

### Internet connection

Test your internet connection by setting up a zoom/meet call with yourself and turn on your camera. This will simulate the exam bandwidth and allow you to see if you have good enough internet. I would suggest do this in parallel of the killer.sh's practice exam. It can tell you if your internet can stream and connect to a remote cluster in a timely manner.

&nbsp;

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

#### Read the requirements by CNCF

https://www.cncf.io/certification/ckad/. Prepare what are required and what are allowed/disallowed.

#### Correct cluster and namespace
4 clusters are provisioned for your exam, and each question provides the command to switch to the correct cluster. Make sure you copy paste and run that command before each question.

Always switch back to `default` namespace if the question does not specify one.

These are the 2 commands I always use after I start a new question. `kn default` and the command to connect to the correct cluster.

#### sudo

I did not have to use `sudo` during the exam despite many people have claiming that some files were blocked.

#### Delete pods

If you make a mistake and want to delete a pod, use `--force --grace-period=0` flags. Deleting a pod takes a long time and you want to save as much time as possible. Using these flag will exit the delete command immediately.

#### use busybox and exec to verify the answer

verify mount setup is correct
```bash
k exec <POD ID> -it -- ls /mount/path
```

verify environment is set up
```bash
k exec <POD ID> -it -- env
```

verify pod/service is up and running.
```bash
k run ping --image=busybox --rm -it --restart=Never -- wget 10.192.0.1 --spider --timeout 5
```

&nbsp;

## Thoughts

Overall, I may have spent around 50 hours spread over 2 months to prepare for this exam. This includes finishing a 30 hours course from linuxacademy and doing the labs by myself. The exam is about time and speed. There are 19 questions and only 2 hours, so you only have about 6 minutes per question. It does not sound too bad but for some questions it is not enough.

After completing the exam, I don't think I will suddenly become a Kubernetes expert. I only took it because I started using it at work. I do think it is a good exam to test yourself if you are familiar with Kubernetes. The real world environment is much more different than the exam and requirements are much more complex. I also think using the `kubectl` at work has helped me.

Trust yourself. You have practiced enough and don't panic. You have a free retry if you fail the first time.

**Good luck 👍**
