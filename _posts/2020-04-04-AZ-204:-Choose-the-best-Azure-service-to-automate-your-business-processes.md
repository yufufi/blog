---
layout: post
title: ! 'Notes: Az-204 - Choose the best Azure service to automate your business processes'
author: yufufi
categories: [technology, programming, certification]
tags: [azure, certification, notes, training]
---

Azure provides the following 4 tech for building workflows.  They all support: `Input -> (Condition | Action)* -> Output`

* **Logic Apps**
	* Design-first | Visual Interface for workflow design
	* Json for workflow design; can be stored in a code repository.
	* 200+ connectors for external services
* **Microsoft Power Automate**
	* Design-first | Visual Interface for workflow design
	* No Dev/IT Pro experience required. 
	* 4 types of flow:
		* Automated: Triggered by external event.
		* Button: Someone needs to press a button.
		* Scheduled: Time triggered.
		* Business process: Implement business processes backed with a [Common Data Service](https://docs.microsoft.com/en-us/power-automate/common-data-model-intro)
	* Built on Logic Apps.
	* Has a mobile app.
	* Includes testing & production environments.
* **WebJobs**
	* Code-first
	* Part of `Azure App Service`
	* Two types:
		* Continuous: Keeps running in a loop.
		* Triggered: Can be started manually or on a schedule.
		* You can write your logic in various programming languages such as Bash, Powershelgl, Php, Python, etc
		* .NET Framework languages can use `WebJobs` SDK to make things easier. e.g. less code needed to interact with Azure App Service. (C# + NuGet only)
	* Pay for the entire VM / App Service Plan that hosts the job.
* **Azure Functions**
	* Code first
	* Bunch of [languages are supported](https://docs.microsoft.com/azure/azure-functions/supported-languages)
	* Hosting infrastructure is completely abstracted out. Automatic scaling based on demand.
	* Templates to get you started. e.g.: ` HTTPTrigger`, `TimerTrigger`, `BlobTrigger`, `CosmosDBTrigger` etc.
	* Pay for consumption only.

	Overall Azure Functions feel superior to WebJobs, except for:
		* Closer control on the event trigger object (see `JobHost`) in WebJobs. (e.g. control retry policies)
		* WebJobs can be integrated to an existing App Service application and managed together.
