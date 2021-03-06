Generating Couchbase APIs with DreamFactory

Modern organizations are under tremendous pressure to efficiently process and incorporate data into every initiative. In fact, the emphasis placed upon extracting, compiling, analyzing, and disseminating information is such that an entirely new field of study known as *data science* has emerged to make sense of it all (for those of you thinking this sounds suspiciously academic, take note Harvard Business Review called data science ["The Sexiest Job of the 21st Century"](https://hbr.org/2012/10/data-scientist-the-sexiest-job-of-the-21st-century).

In addition to formalizing the data management process, IT teams have increasingly embraced heterogeneous database environments and API-driven development in an effort to ensure data can be ingested, processed, and distributed with minimal friction. Gone are the days when an entire organization standardizes upon a particular database vendor; instead, it's now commonplace for enterprises to embrace a mixture of both commercial and open source SQL and NoSQL solutions. 

Of course, there must be some sane way to manage data as it flows through these databases, and that's where APIs come in. Because databases must often be queried from a variety of applications (iOS, web, etc.), developers have in recent years eschewed application-specific libraries in preference for APIs. By creating and plugging into a uniform API, developers can focus on the application itself rather than database-related plumbing.

Or at least that's the idea. The reality is, building and maintaining a database API is an extraordinarily time-consuming and costly endeavor. While an experienced team can build a CRUD API interface in perhaps a few weeks, they often don't take into consideration the many additional requisite features necessary for API management. Capabilities such as authentication and authorization, request volume limiting, caching, and logging are often not considered when planning an API, and figuring out how to implement these capabilities in a piecemeal fashion can be extraordinarily difficult and costly.

Many organizations avoid these headaches by adopting an *API management platform*. One particularly popular such solution is [DreamFactory](https://www.dreamfactory.com). Not only is it open source (commercial versions offering advanced capabilities are also available), but it also includes native support for Couchbase! In this post you'll learn how to generate a Couchbase REST API using DreamFactory in just minutes, and then lock down API access using DreamFactory's role management feature.

#### Introducing DreamFactory

[DreamFactory](https://www.dreamfactory.com/) is an API automation and management framework used by tens of thousands of organizations around the globe. Although its popularity largely stems from the ability to generate full featured, documented, and secure APIs for a variety of SQL and NoSQL databases with no coding required, DreamFactory actually supports thousands of data sources and third-party services, including e-mail delivery providers, mobile notification services including AWS SNS, Apple and GCM Push, and for converting SOAP services to REST.

DreamFactory supports a total of 18 databases, including MySQL, Oracle, Microsoft SQL Server, and... Couchbase! Generating an API is very straightforward, requiring only the provision of a desired API name and Couchbase server credentials. To create an API, you'll begin by logging into the DreamFactory web administration interface. You'll be able to access DreamFactory's key features via the navigational bar:

services-navbar.png

Create the service by clicking on `Services` then `Create`, then choose `Couchbase` from the `Database` category:

service-select.png

Next you'll choose an API name, and set a label and description. The name is particularly important, because as you'll see this will serve as a namespace of sorts for your generated API. The label and description are just for reference purposes within the administration interface:

service-couchbase-config.png

After completing these fields, click on the `Config` tab and supply your Couchbase credentials:

service-couchbase-config2.png

In addition to the `Host`, `Port`, `Username` and `Password` fields, you can optionally cap the number of records returned by a `GET` request, and enable data caching for a set period of time. For the purposes of this tutorial I'll leave those options untouched and just generate the API by pressing the `Save` button.

### Interacting with the API

Once the API has been generated, you can navigate to the `API Docs` interface to experiment with it. DreamFactory generates Swagger documentation for each API, and administrators can provide users with restricted access to this documentation. Select the new API, and you'll be presented with a list of all available endpoints:

api-docs-list.png

Choose the `GET /_schema` endpoint and press the `Try it out` button. Because Swagger documentation is interactive, you can learn how the API works without having to jump straight into code, which can greatly decrease frustration. Pressing `Try it out` will enable the various parameter fields associated with this endpoint:

api-docs-schema.png

NOTE: Because of the breadth of supported databases, and the importance placed on providing a uniform database API interface regardless of the underlying database type, DreamFactory uses generic terms for referring to database-related structures. Logically, these terms don't always map exactly to database-specific terminology. For instance Couchbase uses the term *bucket* to represent a database, whereas DreamFactory uses the generic term *schema* for this representation.

Press the `Execute` button and the API endpoint will be queried, returning all available buckets. Couchbase users will be familiar with the `travel-sample` bucket returned in the following response:

api-docs-schema-response.png

Next, scroll down to the `GET /_table/{table_name}` endpoint to retrieve records (documents) from the `travel-sample` bucket. The process for doing so is similar to that used for retrieving the buckets; you'll select the endpoint, press `Try it out`, and then enter any desired API endpoint parameters. As you can see from the below screenshot, this endpoint supports quite a few parameters:

api-docs-get-table.png

When it comes to selecting records, DreamFactory supports everything you'd typically do when querying a database. You can select specific fields, join tables, filter records, apply limits and offsets, and so forth. For this example let's just keep it simple and scroll down to the bottom of the parameter list and enter `travel-sample` into the `table_name` parameter. Press `Execute` and you'll see the following results:

api-docs-get-results.png

### Generating a Restricted API Key

The API Docs interface is great for learning more about how an API works, however you'll inevitably want to begin querying the API from a web or mobile application. To do so, you'll want to first generate an *API key*. This is because DreamFactory doesn't support the concept of a public API; all APIs are protected by at minimum an API key. Further, each API key is mapped to a *role*. DreamFactory roles determine what it is a client in possession of an API key can do in conjunction with the API. For instance, you might create a role that is read-only, or which might only allow access to a specific database table, or which might only be able to insert records into one table but read records from another table.

To create a role, navigate to the `Roles` tab, and press `Create`. You'll assign a role name and description, and then click the `Access` tab. It's here where the magic happens. In the following screenshot you'll see I've selected the `couchbase` API (service), and set `Component` to `_table/travel-sample/*`, meaning the role can only access this particular data set. Further, I've set `Access` to `GET`, meaning this will be a read-only role:

create-role.png

There are a few other interesting features here, however for the sake of brevity I suggest just pressing `Save` to generate the role.

Next, click on the `Apps` tab to generate an API key. Press the `Create` button and you'll be presented with the following interface:

create-application.png

Here you'll supply an application name and description. You'll also choose a role for this API key, and in this example I've selected the newly generated `couchbase` role. Finally, for the `App Location` I've selected `No storage required` because I'll be connecting to the API from a remote location such as a web application. Press `Save` and you'll be returned to the application key listing:

couchbase-api-key.png

Congratulations, you've just created a restricted read-only API key! Now let's use that key to talk to the Couchbase bucket.

### Connecting to your API

With your API key in hand, it's time to interact with the API from outside of the DreamFactory interface. For the purposes of this example I'll use the [Insomnia REST client](https://insomnia.rest/) however you're free to use [Postman](https://www.getpostman.com/), another API client, or certainly can build out a simple web or iPhone interface. In the following screenshot I've queried the `/api/v2/couchbase/_table/travel-sample` endpoint, and on the right-side of the interface you can see the results:

insomnia.png

Of particular importance here is the `X-DreamFactory-Api-Key` header! It's here where the API key is supplied. Neglecting to supply the key will result in a `400` status code with an error message pertaining to a missing key. Additionally, if this key attempts to access a restricted table or perform an action (insert, update, etc) that hasn't been expressly allowed within the role definition, a `401` unauthorized status code will be returned.

### Resources

Hopefully this introduction to DreamFactory got your mind racing regarding how quickly you can begin building Couchbase-backed applications. If you're interested in learning more the following resources should be useful: 

* [The DreamFactory website](https://www.dreamfactory.com/): The official DreamFactory website includes all kinds of information about the platform.
* [Getting Started with DreamFactory](http://guide.dreamfactory.com/): This recently published guide to DreamFactory fundamentals walks you through key platform capabilities. In particular I suggest reading [chapter 3](http://guide.dreamfactory.com/docs/#chapter-3-generating-a-database-backed-api).
* [DreamFactory Academy](https://academy.dreamfactory.com/): DreamFactory Academy includes several introductory videos. You might additionally want to check out the much more expansive [Youtube channel](https://www.youtube.com/user/dreamfactorysoftware).
* [DreamFactory downloads](https://www.dreamfactory.com/downloads-interstitial/): DreamFactory is available in a wide variety of versions, and is supported on all major platforms. Head to this link to choose your desired version!
* [DreamFactory blog](https://blog.dreamfactory.com/): You'll find a stream of regularly published posts about DreamFactory features here, including this recent post about [creating a geofence API using the Haversine formula, PHP, and DreamFactory's scripted API services](https://blog.dreamfactory.com/creating-a-geofence-api-using-the-haversine-formula-php-and-dreamfactorys-scripted-api-services/).
