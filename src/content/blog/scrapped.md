---
external: false
title: Draft Post
description: This post is a draft and won't be built.
date: 2022-11-22
---

One fine day my friend asked to me help him scrap data from Indiankanoon. Initially, it looked easy as all tasks do before you start them. As we started working on the code, new problems started to arise. We assumed that there were two to three lakh files but to our surprise, there were 2 crore files.

At first, we started scrapping data with basic Python script to open pages and get documents one by one, when we first started scrapping site had Cloudflare security which was rate limiting us to scrapping multiple documents at the same time.
First of all, let's talk about the problems that we faced.
Cloudflare was limiting IPs to open 40-45 documents per minute also was not allowing IPs from outside India. To solve this, we started using multiple proxies to process multiple requests at the same time, but turned out that proxies are slow and it was making it hard to hold a transaction while fetching documents.
We decided to create a service with parallel processing to scrap documents in Python but then we got an idea that we might be able to use docker as each container has a different IP. Once again, We wrote a script to create multiple containers and run each parallelly but that didn’t work either. Because containers were requesting had different IPs requesting to Cloudflare but were using the same response IP of my laptop which Cloudflare was rate limiting.
Of course, We tried using Vercel, render, and other services as well but in the end, all of those services were using IPs outside of India which were not allowed on-site.
After a lot of tries and efforts, we finally decided to use AWS because locally it could take approximately 150 days to scrap even after bypassing Cloudflare and other security.
To use multiple AWS instances we need consistency over all documents hence we decided to create microservices that can be managed by event-driven architecture.
So we created 3 microservices:
To find all document links from the main page
To get document
To get citations for that document
We created a message queue using RabbitMQ to communicate with other services. Service 1 was fetching all document links from the page and publishing them on the queue. Service 2 gets document links from the queue, opens that document, scraps required details, and stores the document in the GCP storage bucket. Meanwhile, Service 3 gets a document and checks if that has been cited in other documents or not.

We stored all this detail in PostgreSQL as well, as we needed those details with links and references with citation