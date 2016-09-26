---
layout: page
title: Cloud Architecture
permalink: /cloud-architecture/
---

<p>While I can cover cloud patterns in general, I mainly focus on patterns that are useful with the Microsoft Azure platform.</p>
<p>The book <a href="https://msdn.microsoft.com/en-us/library/dn568099.aspx" target="_blank">Cloud Design Patterns: Prescriptive Architecture Guidance for Cloud Applications</a>, written by Microsoft, provides solutions for common problems encountered when developing cloud-hosted applications.</p>
<p>It's a great book, by not applied to the Azure platform. I wanted to take some of the pattern I used myself, revisit them with a Azure perspective and provide a implementation.</p>

<h2>External Configuration Store Pattern</h2>
<p>In Azure, there's different deployment models for WebApp, Cloud Service and Virtual Machine (VM). Each deployment model has it's own way of providing application configurations which led to a complexity to manage and update configurations. Some model requires redeploying the application which requires downtime (application restart) and operational times which can be unacceptable. Configuring multi-instance application is even more complex by the fact.</p>
<p>The pattern allow changes at run-time, the centralization and the unification of application configuration management.</p>
<a href="https://jfblierazure.wordpress.com/2016/09/24/external-configuration-store/">Read the article</a>
