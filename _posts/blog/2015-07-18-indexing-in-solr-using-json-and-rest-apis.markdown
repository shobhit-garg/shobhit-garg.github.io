---
layout: post
title:  "Indexing in Solr using JSON and Rest APIs"
date:   2015-07-18
categories: blog
tags: [solr,api]
author: shobhit_garg
share: true
comments: true
excerpt:
---


You can post your documents in solr to index in many ways. XML,JSON,CSV are all the supported formats.Apart from this you can directly link solr to database using DataImportHandler, and execute fullImport to index all the data. As use of JSON and RESTful apis are rising day by day so i thought to update the solr using both of them.

For connectiong Solr to database read : [Connecting Solr to database][connecting-solr-db]

For calling rest-apis , i am using ruby gem [rest-client][rest-client]. You can use any tool or native methods.



To give you all a clear context , these are the fields in my schema.xml.

__schema.xml__

{% highlight ruby %}

<fields>
		<field name='id' type='string' required='true' />
		<field name='name' type='string' multiValued='true' stored='true'/>
		<field name='address' type='text_ws' />
		<copyField source='*' dest='fullText' />
		<field name='fullText' type='text_en'  multiValued='true' indexed='true' 		/>
</fields>

{% endhighlight %}

schema has 4 fields which are id , name , address and fullText. fullText is a copyField so all the content should be copied here automatically.id is the unique field. name is a mutlivalued field so we need to pass a array in json format to update this.



__Sample JSON document to Update:__


{% highlight ruby %}

  {
	        "address" : "Plot No 6, Sector- 22, Noida",
	        "name" : ["The Taj"],
	        "id" : 8753
  }

{% endhighlight %}



For each document you update, you have to put it under keys "add" and "doc" like this way:


{% highlight ruby %}

 final_data = 
 	{"add" : 
 		{"doc" :
		 	{
		        "address" : "Plot No 6, Sector- 22, Noida",
		        "name" : ["The Taj"],
		        "id" : 8753
		    }
	    }
    }

{% endhighlight %}


Format for updating multiple documents:

 {% highlight ruby %}

 final_data = 
 	{"add" : 
 		{"doc" :
		 	{
		        "address" : "Plot No 6, Sector- 22, Noida",
		        "name" : ["The Taj"],
		        "id" : 8753
		    }
	    }

	 "add" : 
 		{"doc" :
		 	{
		        "address" : "Plot No 8, Sector- 24, Delhi",
		        "name" : ["The Oberoi"],
		        "id" : 8754
		    }
	    }
    }

{% endhighlight %}


__Other options:__

There are other options you can send along with document to solr.There should be added on 'add' level. These are:

* commitWithin: 5000   (commit this document within 5 seconds)
* overwrite: false     (default = true . Don't check for existing documents with the same uniqueKey)
* boost: 3.45  (boost factor for document)


{% highlight ruby %}

 final_data = 
 	{"add" : 
 		{"doc" :
		 	{
		        "address" : "Plot No 6, Sector- 22, Noida",
		        "name" : ["The Taj"],
		        "id" : 8753
		    },
		    "commitWithin" : 5000,
		    "overwrite": true

	    }
    }

{% endhighlight %}






#Calling Rest Api to update solr:

{% highlight ruby %}

#Here simple2 is core name.
#url =  "http://localhost:8983/solr/simple2/update" #simple2 is core name.
#json data to update = final_data
#header = {:content_type => 'application/json', :accept => 'application/json'} 

response = RestClient.post("http://localhost:8983/solr/simple2/update", final_data, {:content_type => 'application/json', :accept => 'application/json'})

#response = {"responseHeader":{"status":0,"QTime":20}}"
#status 0 indicating that update is successful.If status is non-zero that means update is not successful.

{% endhighlight %}

After doing this you check for this document in solr, you would not find it. Strange right?

Yes, you have to commit the documents to make them searchable in solr.There are many ways to do this:

* Restart the solr server.
* reload the core.
* Update document with option "commitWithin".(Explained earlier)
* Update document with commit=true
* Execute commit=true after updating multiple documents.


__Update document with commit=true__



{% highlight ruby %}

#commiting document while updating
RestClient.post("http://localhost:8983/solr/simple2/update?commit=true", final_data, {:content_type => 'application/json', :accept => 'application/json'})

#commiting multiple documents after updating
#dummy call to commit all documents
RestClient.post("http://localhost:8983/solr/simple2/update?commit=true", "{}", {:content_type => 'application/json', :accept => 'application/json'})
{% endhighlight %}



#Deleting Document:

{% highlight ruby %}

data = {"delete"  : { "id"  : "8753" }}
RestClient.post("http://localhost:8983/solr/simple2/update?commit=true", data, {:content_type => 'application/json', :accept => 'application/json'})

{% endhighlight %}

For more information , visit [Official Wiki][solr-update-wiki] . 

[rest-client]:   https://github.com/rest-client/rest-client
[solr-update-wiki]:  https://cwiki.apache.org/confluence/display/solr/Uploading+Data+with+Index+Handlers
[connecting-solr-db]: {{ site.url }}/blog/connecting-solr-to-database
