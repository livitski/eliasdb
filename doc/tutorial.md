EliasDB Tutorial
================
The following text will give you an overview of the main features of EliasDB. It shows how to fill the datastore with a simple graph structure and then run queries against this data.

The tutorial assumes you have downloaded and started EliasDB with an empty datastore.

Using the Terminal
------------------
By default EliasDB starts on port 9090 and creates a basic terminal in the webfolder (/web/db/term.html). Assuming you started EliasDB on the locally, point your browser to:
```
https://localhost:9090/db/term.html
```
The generated default key and certificate for https are self-signed which should give a security warning in the browser. After accepting you should see a prompt.t

![](https://github.com/krotik/eliasdb/blob/master/doc/tutorial1.png?raw=true)

You can get an overview of all available command by typing:
```
help
```
Pressing just enter inserts a line break. Pressing Ctrl+Enter submits a command. Run the "info" command and confirm that the datastore is empty.

If you open the browser's development toolbar you can see how the terminal is communicating with the REST API.

Storing Data
------------
The examples in this tutorial are based on an example dataset which contains information on the London tube. To store it type:
```
store
```
into the terminal and press enter. Copy now the contents of the file [tutorial_data.json](https://github.com/krotik/eliasdb/blob/master/doc/tutorial_data.json?raw=true) into the terminal - either drag the file onto the terminal input or copy/paste its contents. 

![](https://github.com/krotik/eliasdb/blob/master/doc/tutorial2.png?raw=true)

After submitting the request you should see a message saying "OK". Running now the "info" should show that the datastore is now filled with data:
```
{ 
    "edge_counts": { 
        "Connection": 354, 
        "StationOnLine": 417 
    }, 
    "edge_kinds": [ 
        "Connection", "StationOnLine" ], 
    "node_counts": { 
        "Line": 13, 
        "Station": 306 
    }, 
    "node_kinds": [ "Line", "Station" ], 
    "partitions": [ "main" ] 
}
```
The datastore is now filled with a simple graph. It has Station nodes which represent tube stations and Line nodes which represent tube lines. The Station nodes are connected via Connection edges to each other and via StationOnLine edges with Line nodes.

The datastore supports partitioning. By default all nodes are stored in the main partition. A query will only be able to see the nodes of the partition it is run against.

To delete data we could run the delete command together with a similar json data structure. For deletion it is enough to only provide the key and kind attributes.

Nodes and edges in the datastore are identified by the kind and the key attribute. A node key needs to unique per kind.

Query data with EQL
-------------------
Data in the datastore can be gueried using EliasDB's own query language called EQL. To get a list of all stored tube lines run the command:
```
get Line
```
The result should be a table of tube lines. Each line having a unique key and a name. 

![](https://github.com/krotik/eliasdb/blob/master/doc/tutorial3.png?raw=true)

We can easily order the table by name by writing:
```
get Line with ordering(ascending name)
```
The main purpose of a graph database is connecting data and form a graph. We can see which stations are on the "Circle Line" by traversing the graph. Run a lookup query:
```
lookup Line "3" traverse Line:StationOnLine:Member:Station end
```
We lookup a single node in the datastore and follow its relationships with Stations. Every successful traversal will add a separate line to the result (i.e. the number of result rows grows exponentially with the number of queried traversals). The relationship is specified in full, we could also omit parts of the traversal spec which would then serve as wildcards. For example to follow all relationships we could write :::. Another way of writing this query would be:
```
get Line where key = "3" traverse Line:StationOnLine:Member:Station end
```
Though, the result is the same, this query is significantly more inefficient. A get query has to go over all nodes and test if they match the where clause while a lookup query can use an efficient direct lookup.

We can now refine the query further by only asking for stations with rail connections:
```
lookup Line "3" traverse Line:StationOnLine:Member:Station where has_rail end
```
To control the data which is displayed we can specify a show clause:
```
lookup Line "3" traverse Line:StationOnLine:Member:Station where has_rail end show Station:name, Station:zone
```
In the last refinement of the search query we can order by name:
```
lookup Line "3" traverse Line:StationOnLine:Member:Station where has_rail end show Station:name, Station:zone with ordering(ascending name)
```
Using the with clause we defined here a post-processing function which sorts the result once all data has been retrieved from the datastore.

For further information on search queries please see the EQL documentation.

Doing a fulltext search
-----------------------
All data which is stored in EliasDB is indexed in a phrase index. The index can be queried from the terminal using the index command. Run "help index" to get an overview of the required parameter. Run now the query:
```
index Station name phrase King's Cross
```
We are looking for a Station which has the name King's Cross in its name. The index can efficiently lookup words, phrases (multiple consecutive words) and attribute values. The index query result is a node key. We can now lookup this node by running:
```
lookup Station "145"
```
![](https://github.com/krotik/eliasdb/blob/master/doc/tutorial4.png?raw=true)
