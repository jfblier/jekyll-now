---
layout: post
title: External Configuration Store
category: Cloud Architecture
tags: Application configuration, Cloud Architecture pattern, Storage
image: cloud-architecture/external-configuration-store/Diagram-External-Configuration-Store.png
excerpt_separator: <!--more-->
---

<p>In Azure, there's differents deployment models for WebApp, Cloud Service and Virtual Machine (VM). Each deployment model has it's own way of providing application configurations which led to a complexity to manage and update configurations. Some model requires redeploying the application which requires downtime (application restart) and operational times which can be unacceptable. Configuring multi-instance application is even more complex by the fact.</p>
<!--more-->

