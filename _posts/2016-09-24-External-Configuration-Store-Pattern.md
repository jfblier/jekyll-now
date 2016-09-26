---
layout: post
title: External Configuration Store
category: Cloud Architecture
tags: Application configuration, Cloud Architecture pattern, Storage
image: cloud-architecture/external-configuration-store/Diagram-External-Configuration-Store.png
date: 2016-09-24 00:00:00
excerpt_separator: <!--more-->
---

In Azure, there's differents deployment models for WebApp, Cloud Service and Virtual Machine (VM). Each deployment model has it's own way of providing application configurations which led to a complexity to manage and update configurations. Some model requires redeploying the application which requires downtime (application restart) and operational times which can be unacceptable. Configuring multi-instance application is even more complex by the fact.
<!--more-->

In a WebApp you need to manage configuration in the Azure Portal or automate it. In a Cloud Service, you provide a .cscfg file while uploading your package. In a VM, you deploy your configuration files with your package.

The pattern allow changes at run-time, the centralization and the unification of application configuration management.
<h2>Motivations</h2>
The <a href="https://msdn.microsoft.com/en-us/library/dn589803.aspx" target="_blank">External Configuration Store</a> pattern allow :
<ul>
	<li>Change configuration at run-time without redeploying</li>
	<li>Centralize configurations by environment (ex.: test, staging, production)</li>
	<li>Unify configuration management into one global approach</li>
	<li>Share configuration between applications and applications instances (if you have multiple instances deployed for the same application)</li>
</ul>
<h2>Solution</h2>
Store configurations in a External Store.

<img class=" size-full wp-image-135 aligncenter" src="{{ site.url }}/images/cloud-architecture/external-configuration-store/diagram-external-configuration-store.png" alt="diagram-external-configuration-store" width="341" height="187" />

As the latency of retrieving values from the External Store could be too high and also because the External Store may not be available during a fraction of a seconds, a Local Cache is used in the application.
<h3>Concerns</h3>
If the configurations are not available your application will not start, this is a single point of failure (SPOF). This is why you should select a technology that has <strong>high-availability</strong>, backed by a <strong>high SLA</strong> and a <strong>redundancy</strong> mechanism.

Do you want to have all configurations mixed in the same store? There's pros and cons, it depends on your needs. For example, sharing configuration can be good for connection string, but it's harder to know which key is used by which application.
<h3>Options</h3>
There's a lot of implementations that you can choose depending if your hosting is in a Cloud or On-Premise platform. As we are using the Azure Cloud, we will describe the Azure Storage and Azure Databases (SqlAzure) options.
<h4>Azure Storage</h4>
The Azure platform offer a <strong>Read Access-Geo Redundant</strong> <a href="https://azure.microsoft.com/en-us/services/storage/?b=16.37">Storage</a> (RA-GRS) that has <a href="https://azure.microsoft.com/en-us/support/legal/sla/storage/v1_0/">99.99% SLA</a> on read. To have this level of SLA you need to select the Replication type <em>Read-access geo-redundant storage </em>:
<p style="padding-left:30px;"><img class="alignnone size-full wp-image-92" src="{{ site.url }}/images/cloud-architecture/external-configuration-store/ga-grs-storage.png" alt="RA-GRS storage" width="281" height="55" /></p>
The cost is really low (pennies per month).

You can use the <strong>Blob</strong> or <strong>Table</strong> to store you configurations. With both approaches you can decide to have the configurations of all your applications into one file/table or to divide them per application. It depends on your needs. A few things to thinks about:
<ul>
	<li>Simplicity to move application configuration between environment
<ul>
	<li>Blob are files, easier to edit and copy/paste.</li>
</ul>
</li>
	<li>Simplicity to update configurations : Who will update the value?
<ul>
	<li>Blob are files which can be updated with any text application</li>
	<li>Table need a special tool to see and edit values.</li>
</ul>
</li>
	<li>Simplicity to query
<ul>
	<li>With the Table, you can use ODATA filters to query the data.</li>
	<li>With the Blob, you can use the Find In Files, but need to download all the files.</li>
</ul>
</li>
	<li>Risk of loosing values (with a single blob file, what will append if somebody in your organisation change the file between the time you get the latest version, update it and but it back?)
<ul>
	<li>With Table, the update is done for a specific value (atomic).</li>
</ul>
</li>
</ul>
<h4>Sql Database (PAAS)</h4>
The creation and maintenance of a replicated Sql Database is heavier. To have a redundant solution, you need to create 2 databases and configure the second database as a secondary database. Azure will replicate the changes automatically after you configured it. If a breakdown occur, you will need to change the secondary database to be the primary, and change it back when the problem is solved.

With only one database, SqlDatabase offers a 99.99% SLA which is really good.

The cost for a Basic tier is around 10$ per month (you need 2 databases if you want the Active Geo-Replication).

It's easy to manage configurations by using the SQL command for querying, creating and updating configurations.
<h2>Implementation using Azure Blob Storage</h2>
The implementation that I will do is with an Azure Storage that will use a Blob Storage. Each application configurations will be stored in a different file in the storage.
<h3>Create the storage</h3>
<blockquote><small><span style="text-decoration:underline;"><strong>Automation</strong></span><strong> - </strong>I normally automate everything with Azure PowerShell, but in this article, as the goal is not automation, I will describe the steps using the Azure Portal as it's simpler, but I encourage you to automate it.</small></blockquote>
In the <strong>Azure Portal</strong>, click <strong>+ New</strong>, under the marketplace, select <strong>Storage</strong>, select <strong>Storage account</strong>. Fill the information required. As we want high-availability, ensure that you select <strong>Read-access geo-redundant storage (RA-GRS)</strong> under the <strong>replication</strong> box. Click the <strong>Create</strong> button. The storage is created.

You need to create the Container under which you will put your configuration files. In the <strong>Azure Portal</strong>, Click <strong>All resources</strong>, <strong>select the newly create storage</strong>. In the <strong>Overview</strong> pane that is shown, under the <strong>Services</strong> section, click on <strong>Blobs</strong>. Click on the <strong>+ Container</strong> button. In <strong>Name</strong>, enter <strong>appconfiguration</strong>. Leave the Access type to private. Click <strong>Create</strong>. After the creation, you should have your container:
<p style="padding-left:30px;"><img class="alignnone size-full wp-image-106" src="{{ site.url }}/images/cloud-architecture/external-configuration-store/appconfiguration-container.png" alt="appconfiguration-container" width="342" height="72" /></p>

<h3>Application configuration file</h3>
Let's create a configuration file named ConfigurationTester.xml that will be used for test:
<pre><code>&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;ApplicationConfigurations&gt;
 &lt;Setting key="ApplicationName" value="ConfigurationTester" /&gt;
 &lt;Setting key="OtherKey" value="23" /&gt;
&lt;/ApplicationConfigurations&gt;
</code></pre>
Upload the file under the Storage Container <em>appconfiguration</em> you created previously.
<blockquote><small><span style="text-decoration:underline;"><strong>How-To</strong></span><strong> - </strong>Use the <strong>Cloud Explorer</strong> in <strong>Visual Studio</strong> or <a href="http://storageexplorer.com/" target="_blank">Microsoft Azure Storage Explorer</a> which is a standalone application.</small></blockquote>
<h3>ApplicationConfigurationManager class</h3>
Let's create the ApplicationConfigurationManager class that will handle reading the external file and give access to the configuration keys/values.
<pre><code>using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using System;
using System.Collections.Generic;
using System.IO;
using System.Xml;

namespace ExternalConfigurationStorePattern
{
    public static class ApplicationConfigurationManager
    {
        public static void Configure(string _storageConnectionString, string _containerName, string _applicationName)
        {
            storageConnectionString = _storageConnectionString;
            containerName = _containerName;
            applicationName = _applicationName;

            Load();
        }

        public static string Get(string key)
        {
            string value;
            if (dict.TryGetValue(key, out value))
                return value;
            return null;
        }

        #region Private Methods

        private static void Load()
        {
            CloudBlobContainer container = GetContainer(storageConnectionString, containerName);
            CloudBlockBlob blob = container.GetBlockBlobReference(applicationName);

            using (var stream = new MemoryStream())
            {
                blob.DownloadToStream(stream);
                stream.Seek(0, SeekOrigin.Begin);

                var doc = new XmlDocument();
                doc.Load(stream);
                if (doc.DocumentElement.Name != "ApplicationConfigurations")
                    throw new Exception("Invalid application configuration file. Root node should be ApplicationConfigurations.");

                var newDict = new Dictionary&lt;string, string&gt;();
                foreach (XmlNode settingNode in doc.DocumentElement.SelectNodes("Setting"))
                {
                    string key = settingNode.Attributes["key"].Value;
                    string value = settingNode.Attributes["value"].Value;
                    newDict.Add(key, value);
                }
                lock (LockObject)
                {
                    dict = newDict;
                }
            }
        }


        private static CloudBlobContainer GetContainer(string storageSetting, string containerName)
        {
            var storageAccount = CloudStorageAccount.Parse(storageSetting);
            var blobClient = storageAccount.CreateCloudBlobClient();
            return blobClient.GetContainerReference(containerName);
        }

        #endregion

        #region Fields

        private static string storageConnectionString;
        private static string containerName;
        private static string applicationName;

        private static Dictionary&lt;string, string&gt; dict = new Dictionary&lt;string, string&gt;();

        private static object LockObject = new object();

        #endregion
    }
}
</code></pre>
You have the basic code that read a configuration file using a Azure Blob Storage. In the application startup, call the Configure method to read the settings of your application. In the code, use the Get method to retrieve the value of a key.

I let the application provide the information on the Storage and the name of the file, that not the responsibility of the ApplicationConfigurationManager to know how to find it, but the application responsibility.

If you decide to implement a unified configuration file, remove the applicationName field.
<blockquote><small><span style="text-decoration:underline;"><strong>Note</strong></span><strong> - </strong>There's no error handling code. That's a good idea to add it if you want to use it in a production environment.</small></blockquote>
<h3>Usage</h3>
Here's the code demonstrating the usage of the class. You need to replace ACCOUNT_NAME and ACCOUNT_KEY with your storage values.
<pre><code>using System;

namespace ExternalConfigurationStorePattern
{
    class Program
    {
        static void Main(string[] args)
        {
            ApplicationConfigurationManager.Configure(
                "DefaultEndpointsProtocol=https;AccountName=ACCOUNT_NAME;AccountKey=ACCOUNT_KEY",
                "appconfiguration",
                "ConfigurationTester.xml");

            Console.WriteLine(ApplicationConfigurationManager.Get("ApplicationName"));
            Console.WriteLine(ApplicationConfigurationManager.Get("OtherKey"));
        }
    }
}
</code></pre>
<h4>Do not cache configuration in application classes</h4>
As the ApplicationConfigurationManager has a internal cache, you do not need to cache values in other classes. This way, if you add auto-update functionality (see Auto-Update section below) later, it will work without other changes.
<h3>Backup your data</h3>
RA-GRS mode Azure take care of the replicating elements to a different DataCenter, but you are not protected against data corruption or file erased. Since configurations are vital information's, you probably want to backup your files outside Azure.

Check the article <a href="https://azure.microsoft.com/en-us/documentation/articles/storage-moving-data/">Moving data to and from Azure Storage</a> to see the options.
<h2>Improvements</h2>
<h3>Auto-Update</h3>
You can improve the solution be adding a task that will run at a specific interval to reload the settings so your application automatically take the changes done in the storage.
<blockquote><small><span style="text-decoration:underline;"><strong>Caution - Fault Tolerance</strong></span><strong> - </strong>Be careful to not replace the settings by a empty dictionary if the file is not available or if the file format is invalid.</small></blockquote>
<blockquote><small><span style="text-decoration:underline;"><strong>Note</strong></span><strong> - </strong>To setup the task you can use a Timer but there's a danger in using it. It is well described in this <a href="http://haacked.com/archive/2011/10/16/the-dangers-of-implementing-recurring-background-tasks-in-asp-net.aspx/">haacked.com post</a>.</small></blockquote>
<h3>Use the .NET ConfigurationManager class</h3>
The ApplicationConfigurationManager is good but there's a drawback, you need to use it everywhere in your applications. If you're developing a new applications that's probably not a problem, but you you have a lots of legacy applications, this change can be a little bit harder.

You can change the implementation so it change the ConfigurationManager settings. There's a good article on <a href="http://andypook.blogspot.ca/2007/07/overriding-configurationmanager.html" target="_blank">Overriding ConfigurationManager</a> described by Andy Pook.
