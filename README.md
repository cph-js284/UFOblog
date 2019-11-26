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

Data in the MongoDB is placed in collections following the structure below  

![diagram2](https://github.com/cph-js284/UFOblog/blob/master/diagrams/diagram2.png)  
  
Obviously both of the databases will be using indexing, this not only makes them faster(performance), but is also a requirement for MongoDB to work with geofunctions. We mention indexing because we feel it is important information, when doing performance testing on databases.  
  
Following MySql columns has been indexed: Locations.name, Locations.Coordinate, BookParts.title, BookParts.author, and all ids.  
The same goes for MongoDB with the following properties: Locations.name, Books.id, Books.title, Location.id, Books.author and Locations.coordinate.  

### Query performance
We will take a look at the above mentioned query  
*(“Given a geolocation, your application lists all books mentioning a city in the vicinity of the given geolocation.”)*  
Below we have included, the query used in our experiment to answer the question in Mysql and MongoDB.  

**Mysql**
```sql
with cities as (
    select *, ST_Distance(ST_GeomFromText(?, 4326), coordinate)/1000 as km_away 
    from Locations where 
    ST_Contains(ST_GeomFromText(ST_AsText(ST_Buffer(ST_GeomFromText(?, 0), ?/111.226)), 4326), coordinate)
)
select distinct BookParts.id, title, part, author,  ST_AsText(coordinate) as point, km_away, name
from cities
inner join BookLocations  on cities.id = BookLocations.location_id
left join BookParts       on BookParts.id = BookLocations.bookparts_id
order by km_away
```
  
*MongoDB*  
```
Locations.aggregate
[ { "$geoNear": {
      "near": {
        "type": "Point",
        "coordinates": [ ?, ? ]
      },
      "distanceField": "distance",
      "maxDistance": ?, // in meters
      "spherical": true,
      "key": "coordinate"
  } },
  { "$unwind": "$booksRef" },
  { "$lookup": {
      "from": "Books",
      "localField": "booksRef",
      "foreignField": "id",
      "as": "Book"
  } },
  { "$project": {
      "Title": "$Book.title",
      "Author": "$Book.author",
      "Part": "$Book.part",
      "Coords": "$coordinate",
      "Population": "$population",
      "City": "$name",
      "DistanceInMeters": "$distance"
  } } ]

```
  
Conducting performance measurement for the above query, and displaying the results in diagrams gives us the results shown below.  
*For a complete table with query measurements please see [artifact link](https://github.com/benjaco-edu/db-guttenburg/blob/master/Artefakt%20Applikationstiming.pdf)*  
  
![diagram3](https://github.com/cph-js284/UFOblog/blob/master/diagrams/diagram3.png)  

Important to note here, is that this diagram depicts averages of several measurements. We see here that MongoDB has an advantage over MySql in performance, when it comes to data containing geospatial information.  

### Findings  
The experiment conducted, shows that with the correct setup, it is possible to achieve a performance increase on data containing geospatial by a factor 2, using MongoDB compared to MySql. 
  
### Perspective
On a grand scale this means that, if you build an application, be it mobile or other, with a heavy reliance on geospatial data stored in a database; you can achieve a performance increase and thereby an overall better application experience by choosing MongoDB over MySql.

### Notes
The entire experiment and construction of the application, containing more than just querying geospatial data can be found [here (danish)](https://github.com/benjaco-edu/db-guttenburg/blob/master/Rapport.pdf).  
