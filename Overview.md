# Overview

Azure Functions opens the door to a style of programming known as serverless, which enables you to develop rapidly, scale out automatically, integrate easily with a host of other Azure services and all the while keep your costs to a minimum. In this course, I'm going to explain everything you need to know to get started with the new version 2 of Azure Functions. 

In particular, we'll create a simple pipeline of C# functions that demonstrates many of the supported triggers and binding types, including interacting with queues and blob storage, as well as sending emails. We'll also see how you can develop in Visual Studio or from the command line with a text editor if you prefer. We'll learn how to automate deployments, as well as how to debug and monitor our functions. By the end of this course, you'll be ready to create, deploy, and manage your own Azure Functions application. And I also hope that you'll have lots of ideas for how you can incorporate them into your own cloud-based applications as I really think they're applicable in a very wide range of scenarios. You'll be able to follow along with this course even if you're never used Azure before. And although I'll be using C# for most of the demos, you can actually use several different languages to create Azure Functions.

## Introducing Azure Functions

At the heart of Azure Functions is the idea of events and code. You simply supply some code, which is just a single function usually written either in C# or JavaScript, and you tell Azure Functions what event should trigger it. 

So for example, you can use Azure Functions to run a function every hour. So the event, or trigger in this case, is a scheduler. And I might use this sort of event trigger to run a nightly batch process that cleans up some data in my database. Another source of event that can trigger a function is new data becoming available. This might be a new message appearing on a queue or a new file being uploaded to Azure blob storage. And I might use this sort of event to trigger sending an email whenever a message appears on a particular queue. Another really useful example of an event that can trigger a function is an HTTP request. Whenever someone calls a specific URL, your function is executed and can respond to that request. So an example might be that I need to handle a web-hook callback from a payment provider like Stripe whenever I sell a product online. But we could also use this trigger type to implement a REST API. 

Now if you've used Azure before or any other cloud provider for that matter, you might be thinking hang on a minute. Can't we already do this? If Azure Functions is about letting your code run in response to event triggers, doesn't Azure already offer multiple ways of achieving exactly the same thing with offerings like virtual machines, web apps, and web jobs? After all, we can already use these to listen to queue events, respond to web-hooks, and do things on a schedule. So why would we need another way? Well, the best way of explaining this is to first consider the other ways of running your code in Azure, and then we'll be in a position to understand why functions are different and why you might want to use them instead. So bear with me for a few minutes. This won't take too long. But let's quickly discuss a few of Azure's existing offerings for running code in the cloud.

## Running Code in Azure

### Virtual machines

Let's start off at the lowest level, which is using virtual machines. In Azure, I can spin up a virtual machine and install anything I want on it. I can install IIS and run an ASP.NET website on it. Or I could create Windows services to do some background work, like processing messages on queues or executing scheduled tasks. And this approach is known as **IaaS**, **Infrastructure as a Service**. And its great benefit is that it gives me complete control of the server. I can install whatever I want on it, including my choice of operating system, and I can fine-tune it to my exact needs, including choosing exactly how much RAM and CPU it has. 

But with this freedom comes several responsibilities. I must ensure that the operating system and the software I'm using is kept patched and up to date. If I want to scale out, I have to manage that myself, deploying additional virtual machines and configuring my own load balancing between them. And this operational overhead comes at a significant cost. You need to have people dedicated to ensuring that your virtual machines in production are running smoothly. So wouldn't it be nice if we could hand that responsibility off to Microsoft and get them to manage our service for us so that we can just focus on writing our application. 

### Azure App Service

Another option for running code in Azure is using what's called Azure App Service. Azure App Service could be described as **Platform as a Service** or **PaaS**. In this model, unlike with virtual machines, the cloud provider takes responsibility for managing and patching the servers. All you need to do is provide the code for your website or background tasks. 

Azure Web Applications run on Azure App Service and are designed to make it super easy to deploy a website into Azure. You simply create a regular website, which might be using ASP.NET Core, but it also supports many of the most popular web development frameworks, including Node.js and PHP. And then you tell Azure to host your website in what's called a hosting plan which gives you the freedom to host multiple websites on a single server for cost-saving purposes or to give your website its own dedicated server and turn on auto-scaling if your web application is in high demand. 

Azure App Service also supports the concept of **web jobs**. Web jobs offer a simple way to get your own background tasks, such as key processing, deployed to the same service in your hosting plan that are running the web application. So what we have with web applications and web jobs is a very programmer-friendly model that makes it super easy to create and deploy multiple websites along with background processing tasks and bundle them all up onto one server to keep your costs down, but with the flexibility to scale up as the demands of your application require. 

Azure Functions is actually built on top of the web jobs SDK and it's hosted on the App Service platform. So in many ways you can think of it as just another part of this same offering, but with a few additional powerful new capabilities that we'll see shortly.

### What's Different About Azure Functions?

Azure Functions, which were originally introduced as part of the App Service platform in 2016 and are now at version 2. What do Azure Functions offer that we can't already do with web applications and web jobs? 

1. Azure Functions offers a simplified programming model. 
  - Creating your first Azure function is trivially easy. 
  - All you have to write is the code that responds to the event you're interested in, whether that's an HTTP request or a queue message for example. 
  - And all the boilerplate code that you'd normally write to connect these events to the code that actually handles them is abstracted away from you, and this makes for a very lightweight and fast-moving style of development where you're really just focused on the code that meets your business requirements and eliminating a lot of repetitive boilerplate code. 

2. Azure Functions offers is a consumption-based pricing model. 
  - With the more traditional Azure offerings we just discussed, virtual machines and web applications, you need at least one server running constantly, and you have to pay for that. 
  - But with Azure Functions, you have the option to select a pay-as-you-go pricing option. 
  - In other words, you only pay when your code is actually running. So if you're listening on a queue and no messages ever arrive, then you pay nothing. 
  - And the Azure Functions framework will automatically scale the number of servers running your functions to meet demand. 
  - So if there's no demand, there might actually be no servers at all actively running your code. But the framework is able to spin one up very quickly if needed. 
  - And so this pricing model can result in dramatic cost savings compared to the more traditional approach of paying a fixed monthly amount to reserve one or more servers.

### Azure Functions Pricing

Like Azure web applications and web jobs, Azure Functions run inside of what's called an App Service Plan. In addition to letting you use some of the existing pricing models for App Service, you also have access to what's called the consumption pricing plan. And here's how the consumption app service plan works. 

You're charged by two metrics
1. How many times your functions run (number of executions)
2. The time your function runs for in seconds multiplied by the RAM allocated. 

So the units of billing are GB seconds. Now the great news is that you get a very generous free grant, which is currently 1 million executions and 400 thousand GB- seconds per month. And this means you can achieve a lot with Azure Functions without paying anything at all. And even when you go over the free limit, the costs are very reasonable. And so with this pricing model, the main way to keep your costs down is by ensuring that when you do run a function, it completes as quickly as possible. And obviously, invoking the functions less frequently and keeping their memory requirements to a minimum will also help. In fact, the consumption plan actually limits function execution time to 5 minutes, and this is a good thing as it prevents costs from accidentally running away if you hit a deadlock in your code. And another nice feature is that you can set an optional maximum daily quota. 

Now you're not forced to use the consumption pricing tier. You can also host Azure Functions apps on a regular App Service Plan, which means you're back to paying for dedicated servers rather than paying for the duration of time your functions run. This makes your monthly costs predictable, and you can also choose from a variety of the pricing tiers that App Service offers depending on your needs. And it also means that you're free from that 5-minute execution time constraint as the length of time a function runs no longer has an impact on your costs. 

And you may also be interested to hear about the premium Azure Functions pricing plan. At the time of recording, this is only available in preview. But hopefully it will be publically available soon. And this offers advanced features like VNet connectivity, as well as improved performance. Of course, the pricing options do change from time to time, so make sure you visit this page on the Azure website, which has details of the pricing of the consumption plan and the free grant limits. And you can configure this page to show prices in your own local currency. Although you need to be aware that even though there is a free and a shared tier, these aren't able to host Azure Functions apps. You'd need to choose the basic pricing tier or higher. 

Finally, the Azure Functions runtime is available as a Docker container. This means that you can run an Azure Functions app on any computer that's able to run Docker, including on-premise datacenters or in other cloud providers. And so obviously if you chose this option, the pricing would simply be whatever you're paying for your container host. So as we've seen, there's a lot of choice about how you host your Azure Functions apps and how you pay for them. 

### Benefits of Azure Functions

So now we're in a position to understand the major benefits of using Azure Functions over the alternative options for hosting our code in Azure. 

First of all, it offers a very rapid and simple development model. In fact, as we're going to see in the next module, you can even create functions by coding directly in the Azure portal. And by eliminating all the boilerplate that you normally have to write just to listen for the event that triggers your function, you can focus instead on writing the code that really matters, the code that responds to that event. 

Second, because it's built on top of the Azure App Service, it comes with a very rich feature set, all of the core stuff from App service including continuous integration, the Kudu portal, Easy Auth, support for SSL certificates and custom domain names, easy management of application settings, and much more are all available to you with Azure Functions. 

Third, as we've just mentioned, there can be dramatic cost savings with the consumption plan as you're only paying for what you use. And this makes Azure Functions a very attractive proposition for startups. You pay almost nothing initially as you're within the free grant. And only when you attract a high volume of users do you need to start paying. 

Finally, this model takes away the need to manage and maintain servers that you have with virtual machines, and it even abstracts away scaling. You simply trust the framework to provide enough servers to meet the needs of your functions, which could be anything from zero to dozens or even hundreds of servers. And that brings us nicely on to the concept of serverless computing.

### Introducing Serverless Architecture

Azure Functions is a serverless platform, or at least it's serverless when you choose the consumption pricing tier. And the concept of serverless has grown rapidly in popularity over recent years, and many cloud providers are offering similar serverless platforms, such as *Amazon's AWS lambda*. 

Now, of course, in one sense the name serverless is a bit silly because of course there are servers needed to run your code. But one of the key ideas behind serverless is that we want to delegate the management and maintenance of our servers to third parties so that we, as developers, can focus exclusively on the business requirements. So in a serverless architecture, you'd rely on **third party platform or Backend as a Service** offerings wherever possible, so for example using Azure Cosmos DB for your database instead of provisioning your own database server on a virtual machine. Or using Auth0 for your logging and authentication rather than hosting your own identity service. 

And there's a growing number of services that meet many of the common needs of modern cloud applications whether that be logging or email sending or search or taking payments. Now, of course, there will still be the need for you to write some of your own custom back end code, and that's where a framework like Azure Functions fits into serverless. 

Instead of provisioning a virtual machine to host your APIs or background processes, you simply tell Azure Functions which events you need to respond to, what to do when those events fire, and let the framework worry about how many servers are actually needed. And this model of lightweight hosting of functions is sometimes referred to as **FaaS** or **Functions as a Service**. 

On its own, Azure Functions isn't a serverless framework. But it fulfills the FaaS part of serverless. And so you could use it in conjunction with other cloud offerings to create an overall serverless architecture. 

### Serverless - A Real-world Example

A while ago, I needed to create a simple website on which I could sell a Windows application I'd written. The website was just some static HTML with a Buy Now button that integrated with a third party payment provider. So far, quite simple. I've got a web server responding to web requests from the browser and responding with HTML, CSS, and JavaScript. But I also needed to handle the web-hook callback from the payment provider when someone bought my product. So I added a web API method to handle that. And that needed to generate a license file and email it out. And so the web server posted messages to a queue. And because I needed somewhere to put the code that listened on the queues, well that ended up on the web server too. 

Before long, there were more and more little bits of code being added. The application let the user report an error, which called a web API. And the application needed to phone home to check that the license was valid. And then I wanted a nightly process to summarize all the activity from each day and email it to me. And so what had started out as a very simple website was growing into a full-blown application with lots of responsibilities. At one point, I was thinking about changing the website to use WordPress to make it easier for me to manage content. But I realized that I couldn't easily make that change now because there was so much custom .NET code that would need a new home if I switched the website over to PHP. 

And this is a familiar story to many developers. Your application starts small and so it only makes sense to have one server. But before you know it, that one server is doing everything, and you've ended up with a **monolithic architecture that's hard to scale**. 

Let's see how serverless could help us in this scenario. Serverless lets us break bits off our monolithic application. 

So let's start by saying that the new purchase web-hook is handled by an Azure function that posts a message onto a queue;
- that message is handled by another Azure function that generates the license and puts it into blob storage. 
- that blob triggers another Azure function that emails it to the customer. 
- The reporting error endpoint is another Azure function that writes a row into Azure Table Storage. 
- The Validate License API is another Azure Function that performs a database lookup of that license code. 
- And the nightly background process is another Azure function triggered by a timer that produces a report and sends it via email. 

Now we've still got our web server, and some serverless advocates might go even further and get rid of that too, replacing it just with static content hosted in blob storage. But even if we don't do that, our web server has now got a lot less to do. And we've managed to decompose our monolith into a set of **loosely coupled functions**. 

You could even think of them as microservices taken to the next level. And like with microservices, we've got freedom on deploy and scale them independently. Now there is one more concept that we need to understand, and that's the idea of function apps. Azure Functions lets you group related functions together into a function app, which allows them to share configuration settings and local resources. So in this application, it would probably make sense to put them all together into a single function app.

### What Are Azure Functions Good For?

You might be wondering what sort of applications are Azure Functions and serverless architecture actually good for? Well, they're not necessarily the right choice for every application. But here's some examples of when they make a lot of sense. 

First of all, they're brilliant for **experiments and rapid prototyping**. It takes just a few minutes to get one up and running. So you can have a prototype application with a functioning back end in next to no time. 

They're also great for automating some of your internal development processes. So for example, you might want to use a web-hook to integrate a slack channel with your continuous integration server. And here it wouldn't make sense to deploy a whole web server just to handle a single web-hook. But with Azure Functions, you just need to write a single function, and you don't need to worry about what server that will run on. 

They're also great for decomposing or extending existing monolithic applications. You can break a bit out, say an existing queue handler into an Azure function. Or you can extend an existing application by adding on a couple of extra functions. And Pluralsight author, Troy Hunt, gave a great example of this a while back in a blog post where he shows how he uses Azure Functions to automate blocking and unblocking users of his web API to avoid server overloads. Just by adding a couple of Azure Functions to his existing architecture, he greatly reduced the load on his web server. 

Azure Functions are also great if you've got a number of queue handlers, but they might all need quite different scaling requirements. When you're managing servers yourself, it can be quite a headache to decide which handlers to group together onto the same server to balance the considerations of cost versus scalability. But with Azure Functions, you can let the runtime solve that for you. 

Another great use case is integrating systems together. Often you need to create intermediate adapters to connect together two systems, and Azure Functions is a really convenient place to do that.

And of course finally, if the whole idea of serverless really appeals to you, then Azure Functions will let you build your entire application that way, composing the whole thing out of lots of loosely coupled functions.

### Module Summary

An Azure function is simply a small piece of code in a language of your choice that runs in response to an event also called a trigger. 

An Azure function app is the unit of deployment and consists of one or more functions. 

Azure Functions is built on top of the existing Azure App Service and offers many of the same features and pricing options that you can use for Azure Web Apps. But it also offers what's called a consumption pricing plan where you pay only for what you use, and it has a generous free monthly grant, which means that during development, you're quite possibly paying nothing at all. 

Serverless describes a style of architecture where the service, scaling, and even wiring up of events are transparently managed for you so you can focus purely on writing the code that solves your business requirements. And Azure Functions doesn't force you to go all in on serverless. In fact, it's very easy to introduce a few Azure functions alongside a more traditional architecture.

They're also great for rapid prototyping. 
