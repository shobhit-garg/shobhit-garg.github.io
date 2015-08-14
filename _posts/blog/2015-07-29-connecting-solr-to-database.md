---
layout: post
title:  "Indexing solr by connecting it to MySql database"
date:   2015-07-29
categories: blog
tags: [solr]
author: shobhit_garg
share: true
excerpt:
comments: true
---



When I was trying to connect solr with database to index the documents directly i was getting different exceptions.I searched on web and found that there were many people who were facing the same type of issues.Finally after trying different approaches i got success in connecting solr to mysql database.

For connecting solr to database there are two jar plugins needed.One is  __solr-dataimporthandler-..\.jar__  and other one is __mysql-connector-java-..\.jar__ e.g. solr-dataimporthandler-extras-5.1.0.jar & mysql-connector-java-5.1.18-bin.jar. First one is used import the data from database and second one is used to connect the solr to mysql database.You can find first one already present in solr repository on location ".../solr-5.1.0/dist/". But you have to add the mysql connector yourself.You can download the connector from [here][mysql-connector] .

After download the connector jar, you need to place it in same dist folder. (You can place it anywhere and update the path of this file in solrconfig.xml but for the purpose of this post i am posting it in the same dist folder.)

Now you have to mention the location of these two jar files in solrconfig.xml . For this you have to add these two lines under the config tag of solrconfig.xml.

{% highlight ruby %}

#mysql-connector-java-5.1.18-bin.jar is the name of file i downloaded from the given url.
#In your case it might be different so please put the right name.
#solr-dataimporthandler-.*\.jar would already be there so you don't need to download it.


<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-.*\.jar" />
<lib dir="${solr.install.dir:../../../..}/dist/" regex="mysql-connector-java-5.1.18-bin.jar" />


{% endhighlight %}


In solrconfig.xml you need to declare the config file location which holds the database connection settings and query.In my case i am naming it as __data-config.xml__ .You can define this in solrconfig.xml using:     

{% highlight ruby %}

<lst name="defaults">
  <str name="config">data-config.xml</str>
</lst>

{% endhighlight %}

Please check the __final solrconfig.xml__ :


{% highlight ruby %}

#filename : solrconfig.xml

<?xml version = '1.0' encoding = 'UTF-8' ?>
<config>
	<luceneMatchVersion>5.0.0</luceneMatchVersion>
  <lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-.*\.jar" />
  <lib dir="${solr.install.dir:../../../..}/dist/" regex="mysql-connector-java-5.1.18-bin.jar" />

	<requestHandler name='standard' class='solr.StandardRequestHandler' default='true'/>
	<requestHandler name='/update' class='solr.UpdateRequestHandler' />
	<requestHandler name='/admin/' class='org.apache.solr.handler.admin.AdminHandlers'/>
	<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
    <lst name="defaults">
      <str name="config">data-config.xml</str>
    </lst>
  </requestHandler>

	<admin>
		<defaultQuery>*:*</defaultQuery>
	</admin>

</config>

{% endhighlight %}


Now you have to create a file __data-config.xml__ and define the connection settings of database there.
Please mention the database_name , user and password accordingly. If you are not hosting mysql on port 3306 then put the url accordingly.
You can use the queries accoring to your schema.For more inforamtion on it please go through [official wiki][dih-wiki] .

{% highlight ruby %}

#file:data-config.xml

<dataConfig>
  <dataSource driver="com.mysql.jdbc.Driver"
     url="jdbc:mysql://localhost:3306/database_name"
     user="..."
     password="..." />
  <document>
    <entity name="person" query="SELECT id,name from persons">
      <field column="id" name="id" />
      <field column="name" name="name" />
   </entity>
  </document>
</dataConfig>

#you have to use queries and field data according to your schema.
#I am using only for the purpose for letting you know the end to end approach.

###################################################


#For my case my final data-config.xml was :

<dataConfig>
  <dataSource driver="com.mysql.jdbc.Driver"
     url="jdbc:mysql://localhost:3306/persons_db"
     user="shobhit"
     password="test" />
  <document>
    <entity name="person" query="SELECT id,name from persons">
      <field column="id" name="id" />
      <field column="name" name="name" />
   </entity>
  </document>
</dataConfig>

{% endhighlight %}



For your convenience i am posting the my __schema.xml__  and __core.properties__ files too. 

{% highlight ruby %}

#filename : schema.xml


<?xml version = '1.0' encoding = 'UTF-8' ?>
<schema name='dihexample' version='1.1'>
	<types>
		<fieldtype name='string' class='solr.StrField'/>
		<fieldtype name='long' class='solr.TrieLongField'/>
		<fieldType name="text_ws" class="solr.TextField" positionIncrementGap="100">
      		<analyzer>
       			 <tokenizer class="solr.WhitespaceTokenizerFactory"/>
      		</analyzer>
    	</fieldType>
	</types>
	<fields>
		<field name='id' type='string' required='true' />
		<field name='name' type='string' multiValued='true' stored='true'/>
		<copyField source='*' dest='fullText' />
		<field name='fullText' type='text_ws'  multiValued='true' indexed='true' 		/>
	</fields>
	<uniqueKey>id</uniqueKey>
	<defaultSearchField>fullText</defaultSearchField>
	<solrQueryParser defaultOperator='OR' />
</schema>

{% endhighlight %}


{% highlight ruby %}

#filename : core.properties

name = dihexample
config = solrconfig.xml
schema = schema.xml
loadOnStartup = true

{% endhighlight %}


__Directory structure of core__

* dihexample/conf/schema.xml
* dihexample/conf/solrconfig.xml
* dihexample/conf/data-config.xml
* dihexample/core.properties


Seems like i have posted the complete core. You shouldn't get any problem while connecting solr to mysql database now.In case you get any problem.Please let me know through comments. I will try to help you.


Now to importing data to solr you can use this command or trigger full-import through solr UI.

{% highlight ruby %}

http://localhost:8983/solr/dihexample/dataimport?command=full-import

{% endhighlight %}

For indexing Solr through json documents please read : [Indexing Solr using json and rest apis][solr-indexing]


[solr-indexing]:     {{ site.url }}/indexing-in-solr-using-json-and-rest-apis
[mysql-connector]:    http://dev.mysql.com/downloads/connector/j/
[dih-wiki]:     https://wiki.apache.org/solr/DataImportHandler



