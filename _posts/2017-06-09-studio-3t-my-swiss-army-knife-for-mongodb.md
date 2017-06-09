---
layout: post
title: "Studio 3T (MongoChef), My "Swiss Army Knife" for MongoDB"
tags: [mongodb, tools]
---

Our production [Worktile](https://worktile.com/) is using MongoDB as backend database. When I joint this team I was looking for a good GUI tool for MongoDB. Very soon I found MongoChef, which named [Studio 3T](https://studio3t.com/) now, and it's being used as my primary MongoDB tool till now.

> Side Note: Anthony Dillon from Studio 3T reached out my blog and found I mentioned Studio 3T in one of my previours post, and he asked me if I can write something about it once he knew I have used Studio 3T for 3 years. And this is one of the reason you saw this post. But the primary reason is, I do think Studio 3T is the best GUI tool for MongoDB and recommend over to you.

> tl; dr: Download [here](https://studio3t.com/download), it's cross-platform and free for 14 days trail. And you can use its non-commercial license for personal projects.

In this post I'm not going to talk its features one by one, since you can find them in their [website](https://studio3t.com/#_product). I would like to mention some highlight points after using it 3 years. How you will say "WOW!" then.

### Create Connection from URI and Generate URI from Connection

You can use Studio 3T connection dialog to create a connection to your MongoDB with specifying the host, port, username and authentication database, etc. and test it. But as a developer you may already know the connection string from your source code. In this case, Studio 3T don't need you to provide your database information separately. You can just copy and page your MongoDB connection string from code to Studio 3T.

Assuming your found the MongoDB connection in the source code like this (in JavaScript)

```javascript
const mongodb = `mongodb://THE_USER_NAME:THE_PASSWORD@192.168.1.100:21201/worktile`;
```

Note that it contains username, password, host, port and the authentication database. Copy the connection string and open Studio 3T. In "New Connection" dialog click the button "From URI..." and paste it.

![001.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/001.png)

Now you will find all connection information were filled automatically based on the connection string.

![002.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/002.png)

![003.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/003.png)

Besides, if you already have a connection saved in Studio 3T and would like to use it in your source code, can just open the connection details window and click "To URI...". Studio 3T will generate connection string from the connection information you specified before. You can also ask Studio 3T to include password in the connection string or not.

![004.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/004.png)

> Studio 3T also supports connect the MongoDB server through SSL, to a replica-set, or even through SSH tunning which means you can use it to touch MongoDB located in a remote server. But my case I only have the rights to connect to local development environment, I don't have the chance to try these powerful features right now.

### 3 Types of Document View: JSON, Tree, Table

MongoDB is a document NoSQL database and it's data structure is JSON-based. So the MongoDB fetch result would a set of JSON. It's very common in other MongoDB tools and the native MongoDB Command-Line Tool. But Studio 3T adds two more views to make us easier and simpler to operate the document.

Below is some documents I found from the "tasks" collection. You can see the result was in tree view. It looks like JSON but every document was collapsed by default, which the "_id" and the properties count.

![005.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/005.png)

You can click the arrow on the left side for each document node to expand. And if it contains array or sub document and can expand them recurrently.

![006.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/006.png)

Tree vew is good, but I would like to show the table view, which I'm using daily and I think the best one. As you know MongoDB is schema-less. Comparing with those relational databases such as MySQL, SQL Server and Oracle, in MongoDB you can push documents in different structure into one collection. But in practice, documents in the same collection may have **very similar** structure. It can be different in some properties but overall it's the same.

Thus, Studio 3T introduces table view as below.

![007.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/007.png)

It looks like a RDB, the collection looks the a table and documents looks like a set of records. Table view allows us to check more information than JSON and tree view. Array and sub documents are presented as below.

![008.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/008.png)

You can double-click on an array or sub document cell, Studio 3T will dig into it can return the content.

![009.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/009.png)

### Bookmarks

Studio 3T allows us to save query into bookmarks. This is useful when I amd developing and testing against some dedicate documents. For example I saved a query which fetch documents for my own user.

![010.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/010.png)

![011.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/011.png)

You can manage bookmarks by clicking the star icon on the top right side. It supports not only the query, but the projection, sort and options as well. You can put all frequent used queries into bookmarks.

![012.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/012.png)

### Import & Export

In [Worktile](https://worktile.com/), not only production data, all log entries are stored in MongoDB. We have a basic web application to search logs. It helps for most cases, but for some complex and strange errors we need to search log entries with complex query, or maybe it needs `aggregate`. Although we can use MongoDB Command-Line tool for these queries but it's inefficiency.

In order to identify the root cause of the error quickly what we can do is to import related log entries to my local MongoDB through and analyze them in Studio 3T.

First of all I will connect to the remote machine to connect to the MongoDB database where log stored. Assuming I want to retrieve log entries for user operation id `1496829900001`. The query command would be

```
db.logs.find({"oid": "1496829900001"});
```

I can export all fetched documents into a JSON file through the command below.

```
mongo --host localhost --port 27017 -u MY_USERNAME -p MY_PASSWORD --authenticationDatabase logs --quite logs --eval 'printjson(db.logs.find({"oid": "1496829900001"}).toArray())' > logs_oid_1496829900001.json
```

Then I can copy this JSON file to my local machine and import them into my local MongoDB through Studio 3T. First connect to my local MongoDB and right-click "Import Collection..."

![013.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/013.png)

Select to format of the import data. Since we exported log entries in JSON so here we select the first one.

![014.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/014.png)

Next, click the green "+" icon to select the JSON file.

![015.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/015.png)

> Besides the file, Studio 3T also allows to import documents from clipboard. This is very useful when importing small amount of documents. For example, fetch documents from remote MongoDB Command-Line and copy the output content, then import them into local machine through Studio 3T directly from clipboard.

You can rename the collection name.

![016.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/016.png)

Finally we can investigate logs from local database.

![017.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/017.png)

### Features I Haven't Try

These are the features I'm using in daily development work. Besides, Studio 3T also provides a lot of cool stuff for database administrator and developers those not familiar NoSQL very well. Below are some of them I didn't try but worth to mention.

**SQL**

I'm ont of many developers are experienced with RDS and SQL, and then moved to MongoDB. At the beginning I need to learn how to write MongoDB query based on SQL query I already to know. For example how to write MongoDB query like `SELECT * FROM my_collection WHERE name LIKE "%shaun%"`, `SELECT name, SUM(score) FROM my_collection GROUP BY name`, etc.

Studio 3T provides a feature where we can write SQL statement and run against MongoDB. In fact, Studio 3T translate SQL into MongoDB query and run it. In fact I do **NOT** want to encourage you to use this feature to query MongoDB, since MongoDB query is something you **MUST** to learn and to use if really want to work with it. But this is a good feature to help you **learn** MongoDB query. You can write SQL query and check how it will be in MongoDB.

![018.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/018.png)

**Aggregate Builder**

Aggregate is a powerful feature MongoDB provides for data mining and analytics. With aggregate we can complete almost all kinds of queries such as filter, projection, cross-collection lookup, de-structure, etc. But it needs more knowledge to write a proper aggregate query.

Studio 3T provides a GUI builder to create aggregate statement. You can find pipeline you want, specify the query content and run it immediately.

![019.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/019.png)

Again, this feature can helps you to understand aggregate. But I recommend you to write aggregate by yourself, and I think this is a **MUST HAVE** skill for MongoDB.

**DBA Related**

Studio 3T also provides many feature for database administrator. You can manage the database users and roles. You can also compare documents between collections and databases. Besides, you can use Studio 3T to analyze a collection and generate the schema information.

Since I'm not a DBA so I don't have the chance to try them.

### Customer Support

Besides these cool feature I mentioned below, Studio 3T also provides very positive customer support. Their support team gave a very deep impression when I reported a bug in v4.3. When I upgraded to v4.3 I found the application crashed when I edited a document in the edit dialog. I just sent a feedback and I keep working.

Several hours later I got an email sent from Studio 3T support team to check the error I reported. I provides more information about that bug.

![020.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/020.png)

Several days later I received another email from Studio 3T. They just released a new version which contains the fix of the bug I reported. To me, this means Studio 3T respects every users, tracks every feedback and provides updates as quick as possible.

![021.png]({{site.baseurl}}/img/2017-06-09-studio-3t-my-swiss-army-knife-for-mongodb/021.png)

I believe Studio 3T will be success because of not only the features, but their attitude.

### Summary

Studio 3T is the only tool I'm using for MongoDB tool. Actually I use it everyday. You can try it from the [download page](https://studio3t.com/download). Hopes you love it.