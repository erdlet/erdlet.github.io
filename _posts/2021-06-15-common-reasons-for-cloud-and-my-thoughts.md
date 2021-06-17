---
layout: post
title:  "Common reasons for introducing cloud technologies and what I think about them"
date:   2021-06-16 09:00:00 +0200
categories: deployment,infrastructure,personal-thoughts
author: Tobias Erdle
---

It seems to me that the IT world has been revolving around container and cloud technologies for a long time now. Sometimes I even think that cloud technologies are considered by some companies to be the Holy Grail that solves all problems of their existing infrastructure and software. In this context I keep hearing or reading the same reasons for adopting cloud technologies. However, since I believe that these reasons often fail to address the real problem, I will now explain what I believe to be the real problems and how I think they can be solved better. Be warned: sarcasm, cynicism and rants may appear in places. The opinion expressed is 100% my personal view of things.

## What is the common context of the companies I'm talking about?

In this post I'll assume medium sized companies which don't have a software based end user product which is consumed by masses of customers. Or heavy load by other computational tasks. Most of their applications are used internally or only for a containable number of users where increased usage loads are predictable.

## Reason 1: The cloud helps us to achieve high availability!

Many companies want their software to be "highly available" by means of cloud technologies. They see this as unfeasible with conventional methods because they are difficult to scale, switching to a replacement system in the event of an error or deployments take too long and even planned downtimes are too high. It's true that cloud technologies can help to solve these issues. However, I highly doubt that the real problems are located in the infrastructure.

Let's take deployments as an example. I've seen many deployments, from very large to very small applications. Their duration is rarely impeded by the underlying infrastructure. There were usually two key factors involved: processes and bloated software artifacts. What do I mean by that? I have spent many evenings in the office to get the deployment of a single Java WAR file done. It wouldn't actually be complicated, yet it took hours because the processes around it involved many manual steps. Accordingly, the application was not available for a long time. Funnily enough, these processes are then softened for deployments into a cloud environment, for example, because otherwise one would be as slow as before. The fact that the deployment could also be made easier beforehand through other processes is usually not taken into account.

Now to the second point: bloated deployment artifacts. In the Java world exists actually a nice separation of runtime environment and application code. Actually. Due to various circumstances, people went away from very lightweight WAR files and replaced them with fat jars or other big deployment artifacts, which led to application startup times getting longer and longer. For myself I like to use classic application servers, which I need to configure once and then deploy applications to it that communicate with the runtime environment exclusively via APIs. And what should I say... it's really fast. But if I use certain other technologies, the startup time and thus the deployment becomes slower with every connection to a third-party system like databases: unlike the classic approach, the application has to rebuild connection pools etc. every time it gets deployed, which takes its time.

So if you want to achieve higher availability, first check your processes and software before you think about using cloud technologies. I'm sure you'll find something to improve that costs less than introducing a lot of new technologies.

## Reason 2: We can easily scale our applications!

Another argument often cited in favour of cloud technologies is "scalability". What is meant here is horizontal scalability, i.e. the use of multiple instances of the same application. There is no denying that this is very easily done with the help of Docker and Co.

But when I hear this argument in the context described at the beginning, I have to ask myself two things:

1. Why does an application with a very precisely definable load need to be flexibly scalable?
2. How is the application supposed to scale? It uses a fixed session!

On the first point, it can be said that most applications I have seen at enterprises simply do not need to be horizontally scalable. Most of them have a manageable number of parallel users, no performance-intensive tasks to process, and certainly no sudden load spikes. In this case, the argument made is simply the unreflected repetition of marketing slogans, which are certainly no help.

The second point is more interesting: the use of sessions. Many applications use sessions to provide data of the current user. However, in the standard case, these have the property that they are firmly bound to a server instance. Scaling this is very costly and actually something you don't want to do. However, cloud technologies will not help me here either, because the real problem stays the same. If you then replace server side sessions with other solutions, the first point usually occurs again.

Therefore I think that the argument of scalability is only valid in very special cases, e.g. when you have a public application with a lot of users or intensive batch tasks which can be processed in parallel.

## Reason 3: Cloud technologies make everything easier!

Simplicity is often cited as another reason for adopting cloud technologies. It is easier to update software such as databases, it is easier to deliver software because you can directly bring the runtime environment as a container image, ... For simple use cases or third party software this may absolutely be true. For example, I like to use Docker myself to deploy local test databases or setting up a Wordpress environment.

But these are absolutely trivial use cases. As soon as I want to build different test and production environments for self-made software, I very quickly need more software than just Docker. Then I run into products like Kubernetes, Rancher, etc. which are a world unto themselves. If I then also host these myself, stable operation must be ensured so that the corresponding availability exists. Wait, wasn't high availability an argument for adopting cloud technologies? I think you can see very well here that there is more to it than just the cloud technology itself. A single-node Kubernetes cluster can collapse just as easily as a virtual server with an application server. But in my opinion, this is exactly what is often ignored or overlooked.

Let's move on to the deployment of containerized applications. With some recent technologies and programming languages it is definitely incredibly easy to build images for container environments. But not everywhere you can or want to use these technologies. When this happens, the implementation and deployment of the application becomes much more complex. For example, if you try to run a Jakarta EE application in a Docker container using a "normal" application server, the build of the image can very quickly end up in complicated Dockerfiles. Here I also think that it would be easier to stick to known infrastructures, because they are usually better tailored to the characteristics of the operating model of my respective ecosystem and therefore easier to control.

## Reason 4: "This is how it's done today!" or "But XYZ is using this technology too!"

Ah. My favorite "arguments". I don't know how many times I've heard these in discussions.

So seriously? Just saying "That's how it's done today" or "But Amazon / Netflix / ... do it that way" is probably the most unsubstantiated argument you can make. But let's take it one step at a time...

**"But that's how it's done today"** ignores the fact that every company, indeed every software project is different. The features that cloud technologies bring with them are not needed everywhere and only lead to higher expenses. Why should I introduce a highly scalable system for software that is used by only a few people and that is rarely redeployed, which in the end devours a lot of budget? Good questions, but they are practically never answered in such discussions.

**"But Amazon / Netflix / ... do it that way too"**

Dear small / mid-sized / whatever company, you are NOT Amazon, Netflix or Google. Your software does NOT have to abruptly process thousands or millions of unforeseen requests. So you do NOT need to start spending a lot of time and money on implementing unnecessary technologies. It's better for everyone, really! Thanks.

## Conclusion... or something I think it is

Now that I have talked about a few points, the impression might have been created that I reject all cloud technologies and would prefer to return to the digital stone age. This is not the case, of course.

I think that all technologies have their right to exist. Cloud technologies are great for fulfilling most of the above-mentioned points and should be used for this purpose as soon as the reasons really apply. And that's my final conclusion: all of the reasons, except reason #4, for adopting cloud technologies are good, if they actually apply. So if a startup or midsize company has a need for high scalability, etc., it should take advantage of the cloud. All others are better off concentrating on their strengths and sticking to familiar approaches, which will save a lot of time and money.
