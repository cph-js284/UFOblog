# UFO Blog Assigment

By:  
Benjamin Schultz Larsen     cph-bl135@cphbusiness.dk  
Jacob Sørensen              cph-js284@cphbusiness.dk  
Yosuke Ueda                 cph-yu173@cphbusiness.dk  
  
# The usage of geospatial data in databases
<p>
Geospatial coordinates is something that has seen a dramatic increase in use since the introduction of mobile devices.  
There is a noticeable difference in how well different types of databases process this type of data.  
In fact, choosing the correct type of database will lead to a factor 2 increase in database performance.  
Consequently leading to your database being able to handle twice the amount of request in the same timeframe.
  
Do note; that this blog will not be focused specifically on mobile device but rather geodata which is prominently used by such devices.  
</p>
<p>
In the following blog we will investigate which type of database is best suited for handling geospatial data. There are many different types of databases to choose from; graph-databases like NEO4J, key/value stores like Redis, document databases like MongoDB and relational databases like MySql and Mssql.  
Conducting experiments on all of these different types of databases is a task too large for the scope of this blog - so we have decided to narrow the scope to just experimenting using the MongoDB and MySql databases - a non-relational-database versus a relational database; which of these 2 types of databases will handle geospatial in the most performant way (response time)  
  
*MySql has considerably more built-in methods for handling geospatial data compared to  MongoDB, but how would it perform/hold up compared to MongoDB?*
  
[MySql geofunctions](https://dev.mysql.com/doc/refman/5.7/en/spatial-function-reference.html)  
[MongoDB geofunctions:](https://docs.mongodb.com/manual/geospatial-queries/#id1)  

### The Experiment
Which database can in the most performant way get a list of geolocation, closest to a location defined by the user. This use case is seen in several different applications i.e. Just Eat’s “restaurants near you” and Facebook’s “event near you”.  
  
  
### Regarding data
We will use a list of city names where the population of the city is 15.000 or more. This is roughly about 23.000 cities worldwide. The data can be found [here](https://raw.githubusercontent.com/benjaco-edu/db-guttenburg/master/cities15000.txt). We will couple this data with every city mentioned in every single book written in English that is no longer under  copyright protection; this amounts to roughly 37.000 books - available at the Project Gutenberg site [gutenberg.org](http://www.gutenberg.org/)  
  
Specifically, we will measure the response time for each database executing the following query:
  
“Given a geolocation, your application lists all books mentioning a city in the vicinity of the given geolocation.”  
*- this query is part of the assignment from which this experiment is based on*  
  
The above mentioned data will be placed in the MySql database using the following schema (3rd normal form)

![diagram1](https://github.com/cph-js284/UFOblog/blob/master/diagrams/diagram1.png)  

Data in the MongoDB is placed in collections following the structure below  

![diagram2](https://github.com/cph-js284/UFOblog/blob/master/diagrams/diagram2.png)  
MongoDb is document based and do not impose any schema. If one decides to store all the data in one single collection, it performs the reading and updating operations fast as the data is embedded in one document. This embedded data model will however contain various redundant data as the citynames will be stored every time it is mentioned in the books. This will not necessary become a scalability issue for MongoDb, but we might expect an increase in citynames mentioned in future books or in extensive tour guides.  
If one decides to split the data into 2 collections, it needs to update both collections for books and city every time a book is added. Moving to this form will replace the embedded structure with additional field which serves as the reference between the 2 collection. By creating this relation, we avoid the data redundancy and can access the lists by reference
    
Obviously both of the databases will be using indexing, this not only makes them faster(performance), but is also a requirement for MongoDB to work with geofunctions. We mention indexing because we feel it is important information, when doing performance testing on databases.  
  
(MySql refenrece)[https://dev.mysql.com/doc/refman/5.5/en/optimization-indexes.html]

Following MySql columns has been indexed: Locations.name, Locations.Coordinate, BookParts.title, BookParts.author, and all ids.  
The same goes for MongoDB with the following properties: Locations.name, Books.id, Books.title, Location.id, Books.author and Locations.coordinate.  
  
for further comparison between index and no index; please see page 13, query 4 (Q4) where UI is without index and MI is with an index (Link to report)[https://github.com/benjaco-edu/db-guttenburg/blob/master/Rapport.pdf ]  
  
### Query performance
Below we have included, the query used in our experiment to answer the question in Mysql and MongoDB.  

**Mysql**
```sql
with cities as (
    select *, ST_Distance(ST_GeomFromText(<coordinate as point>, 4326), coordinate)/1000 as km_away 
    from Locations where 
    ST_Contains(ST_GeomFromText(ST_AsText(ST_Buffer(ST_GeomFromText(<coordinate as point>, 0), <distance from point in km>/111.226)), 4326), coordinate)
)
select distinct BookParts.id, title, part, author,  ST_AsText(coordinate) as point, km_away, name
from cities
inner join BookLocations  on cities.id = BookLocations.location_id
left join BookParts       on BookParts.id = BookLocations.bookparts_id
order by km_away
```
  
st_distance return a distance between two points in meters  
st_contains returns a boolean indicating whether or not a point is inside or not inside a defined area  
st_buffer creates and area around a givin coordinate or area around a defined(geo-object) area. Works only in SRID 0  
st_GeomFromText,st_AsText convert back and forth between “normal” text and the internal representation of a mysql coordinate   
  
The only reason we convert back and forth is to get a buffer around the specific point, due to the fact the st_buffer can not work with SRID 4326 (the globe), however we only we only used the buffer as a rough filtering tool to distinguish between inside and outside of the selected area.
Mysql can use its own index when using st_contains to see if a point is inside the buffer, when measuring the distance to each point, then the profile tells us that it has been going through all the rows - with the buffer, it does not.

As an aside we will note that we actually ran the code both with and without the buffering and the result was almost identical (within a 2% margin).

**MongoDB**  
```
Locations.aggregate
[ { "$geoNear": {
      "near": {
        "type": "Point",
        "coordinates": [ <longitude>, <latitude> ]
      },
      "distanceField": "distance",
      "maxDistance": <meters from point>, 
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
  
$geoNear filters away objects there isn’t near a given point, and attaches the distance the the returned object in the same operation.
  
Conducting performance measurement for the above query, and displaying the results in diagrams gives us the results shown below.  
*For a complete table with query measurements please see query 4 in [artifact link](https://github.com/benjaco-edu/db-guttenburg/blob/master/Artefakt%20Applikationstiming.pdf)*  
  
![diagram3a](https://github.com/cph-js284/UFOblog/blob/master/diagrams/diagram3a.png)  

It is important to note here; that this diagram depicts averages of 5 measurements&ast;, see “Notes”-section below for link to relevant data. We see here that MongoDB has an advantage over MySql in performance, when it comes to data containing geospatial information.

&ast;*These measurements are in fact so close to each other, that the standard deviation is below 0.2% for the MySQL measurement and approx. 0.5% for the MongoDB measurement.*

### Findings  
The experiment conducted, shows that with the correct setup, it is possible to achieve a performance increase on data containing geospatial by a factor 2, using MongoDB compared to MySql. 
  
### Perspective
On a grand scale this means that, if you build an application, be it mobile or other, with a heavy reliance on geospatial data stored in a database; you can achieve a performance increase and thereby an overall better application experience by choosing MongoDB over MySql.  
Reason for mongodb handling geodata with better performance compared to mysql might be because; mongoDB is finding the relevant geopoint directly using its index, and from here, expands the search area until the max distance is found, this makes mongodb able to stream the data back as well, and return the first result very fast.
For more in depth please read the following (link)[https://www.mongodb.com/blog/post/geospatial-performance-improvements-in-mongodb-3-2]


### Notes
The entire experiment and construction of the application, containing more than just querying geospatial data can be found [here (danish)](https://github.com/benjaco-edu/db-guttenburg/blob/master/Rapport.pdf).  
