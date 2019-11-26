# UFO Blog Assigment

By:  
Benjamin Schultz Larsen     cph-bl135@cphbusiness.dk  
Jacob Sørensen              cph-js284@cphbusiness.dk  
Yosuke Ueda                 cph-yu173@cphbusiness.dk  
  
# The usage of geospatial data in databases
<p>
Geospatial coordinates is something that has seen a dramatic increase in use since the introduction of mobile devices.
There is a noticeable difference is how well different types of databases processes this type of data.
In fact, choosing the correct type of database will lead to a factor 2 increase in database performance.
Consequently leading to a higher throughput of customers for your application.
</p>
<p>
In the following blog we will investigate which type of database are best suited for handling geospatial data. There are many different types of databases to choose from; graph-databases like NEO4J, key/value stores like Redis, document databases like MongoDB and relational database like MySql and Mssql.   

Conducting experiments on all of these different types of databases is a task too large for the scope of this blog - so we have decided to narrow the scope to just experimenting using the MongoDB and MySql databases - a non-relational-database versus a relational database; Which of these 2 types of databases will handle geospatial the in the most performant way (response time)</p>

*MySql has considerably more built-in methods for handling geospatial data compared to  MongoDB, but how would it perform/hold up compared to MongoDB?*
  
  

[MySql geofunctions](https://dev.mysql.com/doc/refman/5.7/en/spatial-function-reference.html)
[MongoDB geofunctions:](https://docs.mongodb.com/manual/geospatial-queries/#id1)

### The Experiment
To discover the difference in performance between the 2 databases we will conduct the following experiment: Measure the response time for each database executing the following query:

*“Given a geolocation, your application lists all books mentioning a city in the vicinity of the given geolocation.”*



### Regarding data
We will use data on cities containing 15k+ population.This is roughly about 23.000 cities. The data can be found [here](https://raw.githubusercontent.com/benjaco-edu/db-guttenburg/master/cities15000.txt) and contains all 15k+ pop of city data. We will couple this data with every city mentioned in every single book, in english which is no longer under copyright, which amounts to roughly 37.000 books - available at the project gutenberg site [gutenberg.org](http://www.gutenberg.org/)<br>
<br>
The above mentioned data will be placed in the MySql database using the following schema (3rd normal form)

![diagram1](https://github.com/cph-js284/UFOblog/blob/master/diagrams/diagram1.png)
