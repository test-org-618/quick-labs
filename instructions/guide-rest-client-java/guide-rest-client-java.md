
# Consuming a RESTful web service
## What you'll learn
artists.json

You will learn how to access a REST service, serialize a Java object that contains a
list of artists and their albums, and use two different approaches to deserialize
the returned JSON resources. The first approach consists of using the Java API for JSON Binding (JSON-B)
to directly convert JSON messages into Java objects. The second approach consists of
using the Java API for JSON Processing (JSON-P) to process the JSON.

The REST service that provides the artists and albums resources has already been written
for you and is accessible at the following link when the server is running 
```
curl http://localhost:9080/artists
```
{: codeblock}

, which responds with the [hotspot]`artists.json`.

You will implement the following two endpoints using the two deserialization approaches:

* **.../artists/total** to return the total number of artists in the JSON
* **.../artists/total/<artist>** to return the total number of albums in the JSON
for the particular artist

If you are interested in learning more about REST services and how you can write them, read
[Creating a RESTful web service](https://openliberty.io/guides/rest-intro.html).



# Getting Started

If a terminal window does not open navigate:

> Terminal -> New Terminal

Check you are in the **home/project** folder:

```
pwd
```
{: codeblock}

The fastest way to work through this guide is to clone the Git repository and use the projects that are provided inside:

```
git clone https://github.com/openliberty/guide-rest-client-java.git
cd guide-rest-client-java
```
{: codeblock}

The **start** directory contains the starting project that you will build upon.



## Try what you’ll build
The **finish** directory in the root of this guide contains the finished application. Give it a try before you proceed.

To try out the application, first navigate to the finish directory and then run the following Maven goal to build the application and run it inside Open Liberty:
```
cd finish
mvn install liberty:start-server
```
{: codeblock}



You can find your service at http://localhost:9080/artists.

Now, you can access the endpoint at http://localhost:9080/artists/total to see the total number of artists, and you can access the endpoint at **\http://localhost:9080/artists/total/<artist>** to see a particular artist`s total number of albums.

After you are finished checking out the application, stop the Open Liberty server by pressing `CTRL+C` in the shell session where you ran the server. Alternatively, you can run the `liberty:stop` goal from the `finish` directory in another shell session:

```
mvn liberty:stop
```
{: codeblock}
After you are done checking out the application, stop the Open Liberty server:

```
mvn liberty:stop-server
```
{: codeblock}


# Starting the service

Navigate to the **start** directory to begin.

```
cd start
```
{: codeblock}



This guide is already setup with a general application. As you progress through the guide,
you will update the code directly and automatically see the results by using development mode.

Start Open Liberty in development mode, which starts the Open Liberty server and listens for file changes:

mvn liberty:dev

When the server is running, you can find your service at the 
```
curl http://localhost:9080/artists
```
{: codeblock}

 URL.

# Creating POJOs

Artist.java

Album.java

To deserialize a JSON message, start with creating Plain Old Java Objects (POJOs) that represent what
is in the JSON and whose instance members map to the keys in the JSON.

For the purpose of this guide, you are given two POJOs.
The **Artist** object has two instance members **name** and **albums**, 
which map to the artist name and the collection of the albums they have written. The **Album** object represents a 
single object within the album collection, and contains three instance members **title**, **artistName**, and **totalTracks**, which map to the album title, the artist who wrote the album, and the number of tracks the album contains.

# Introducing JSON-B and JSON-P

JSON-B is a feature introduced with Java EE 8 and strengthens Java support for JSON.
With JSON-B you directly serialize and deserialize POJOs. This API gives you a
variety of options for working with JSON resources.

In contrast, you need to use helper methods with JSON-P to process a JSON response.
This tactic is more straightforward, but it can be cumbersome with more complex classes.

JSON-B is built on top of the existing JSON-P API. JSON-B can do everything that JSON-P can do
and allows for more customization for serializing and deserializing.

### Using JSON-B
Artist.java

JSON-B requires a POJO to have a public default no-argument constructor for deserialization
and binding to work properly.

The JSON-B engine includes a set of default mapping rules, which can be run without
any customization annotations or custom configuration. In some instances, you might
find it useful to deserialize a JSON message with only certain fields, specific
field names, or classes with custom constructors. In these cases,
annotations are necessary and recommended:

* The **@JsonbProperty** annotation to map JSON keys to class instance members and vice versa.
Without the use of this annotation, JSON-B will attempt to do POJO mapping, matching the keys in
the JSON to the class instance members by name. JSON-B will attempt to match the JSON key 
with a Java field or method annotated with **@JsonbProperty** where the value in the
annotation exactly matches the JSON key. If no annotation exists with the given JSON key, 
JSON-B will attempt to find a matching field with the same name. If no match is found, 
JSON-B attempts to find a matching getter method for serialization or a matching setter 
method for de-serialization. A match occurs when the property name of the method matches 
the JSON key. If no matching getter or setter method is found, serialization or 
de-serialization, respectively, fails with an exception. The Artist POJO does not require 
this annotation because all instance members match the JSON keys by name.

* The **@JsonbCreator** and **@JsonbProperty** annotations to annotate a custom constructor.
These annotations are required for proper parameter substitution when a custom constructor is used.

* The **@JsonbTransient** annotation to define an object property that does not map to a JSON
property. While the use of this annotation is good practice, it is only necessary for serialization.

For more information on customization with JSON-B, see the [official JSON-B site](http://json-b.net).


# Consuming the REST resource

Artist.java

Album.java

Consumer.java

The **Artist** and **Album** POJOs are ready for deserialization. 
Next, we'll learn to consume the JSON response from your REST service.

Create the `Consumer` class.
```
touch src/main/java/io/openliberty/guides/consumingrest/Consumer.java
```
{: codeblock}


> [File -> Open]guide-rest-client-java/start/src/main/java/io/openliberty/guides/consumingrest/Consumer.java
```

package io.openliberty.guides.consumingrest;

import java.util.List;
import java.util.stream.Collectors;

import javax.json.JsonArray;
import javax.json.JsonObject;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.Response;

import io.openliberty.guides.consumingrest.model.Album;
import io.openliberty.guides.consumingrest.model.Artist;

public class Consumer {
    public static Artist[] consumeWithJsonb(String targetUrl) {
      Client client = ClientBuilder.newClient();
      Response response = client.target(targetUrl).request().get();
      Artist[] artists = response.readEntity(Artist[].class);

      response.close();
      client.close();

      return artists;
    }

    public static Artist[] consumeWithJsonp(String targetUrl) {
      Client client = ClientBuilder.newClient();
      Response response = client.target(targetUrl).request().get();
      JsonArray arr = response.readEntity(JsonArray.class);

      response.close();
      client.close();

      return Consumer.collectArtists(arr);
    }

    private static Artist[] collectArtists(JsonArray artistArr) {
      List<Artist> artists = artistArr.stream().map(artistJson -> {
        JsonArray albumArr = ((JsonObject) artistJson).getJsonArray("albums");
        Artist artist = new Artist(
          ((JsonObject) artistJson).getString("name"),
          Consumer.collectAlbums(albumArr));
        return artist;
      }).collect(Collectors.toList());

      return artists.toArray(new Artist[artists.size()]);
    }

    private static Album[] collectAlbums(JsonArray albumArr) {
      List<Album> albums = albumArr.stream().map(albumJson -> {
        Album album = new Album(
          ((JsonObject) albumJson).getString("title"),
          ((JsonObject) albumJson).getString("artist"),
          ((JsonObject) albumJson).getInt("ntracks") );
        return album;
      }).collect(Collectors.toList());

      return albums.toArray(new Album[albums.size()]);
    }
}
```
{: codeblock}



### Processing JSON using JSON-B
Consumer.java

pom.xml

JSON-B is a Java API that is used to serialize Java objects to JSON messages and vice versa.

Open Liberty's JSON-B feature on Maven Central includes the JSON-B provider through transitive dependencies.
The JSON-B APIs are provided by the MicroProfile dependency in your **pom.xml** file. 
Look for the dependency with the **microprofile** artifact ID. 

The **consumeWithJsonb()** method in the **Consumer** class makes a **GET** request to the
running artist service and retrieves the JSON. To bind the JSON into an **Artist**
array, use the **Artist[]** entity type in the **readEntity** call.

### Processing JSON using JSON-P
Consumer.java

The **consumeWithJsonp()** method in the **Consumer** class makes a **GET** request
to the running artist service and retrieves the JSON. This method then uses the
**collectArtists** and **collectAlbums** helper methods. These helper methods will
parse the JSON and collect its objects into individual POJOs. Notice that you can
use the custom constructors to create instances of **Artist** and **Album**.

# Creating additional REST resources
Consumer.java

ArtistResource.java

Now that you can consume a JSON resource you can put that data to use.

Replace the `ArtistResource` class.

> [File -> Open]guide-rest-client-java/start/src/main/java/io/openliberty/guides/consumingrest/service/ArtistResource.java



```
package io.openliberty.guides.consumingrest.service;

import javax.json.JsonArray;
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.UriInfo;

import io.openliberty.guides.consumingrest.model.Artist;
import io.openliberty.guides.consumingrest.Consumer;

@Path("artists")
public class ArtistResource {

    @Context
    UriInfo uriInfo;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public JsonArray getArtists() {
    	return Reader.getArtists();
    }

    @GET
    @Path("jsonString")
    @Produces(MediaType.TEXT_PLAIN)
    public String getJsonString() {
      Jsonb jsonb = JsonbBuilder.create();

      Artist[] artists = Consumer.consumeWithJsonb(uriInfo.getBaseUri().toString() +
        "artists");
      String result = jsonb.toJson(artists);

      return result;
    }

    @GET
    @Path("total/{artist}")
    @Produces(MediaType.TEXT_PLAIN)
    public int getTotalAlbums(@PathParam("artist") String artist) {
      Artist[] artists = Consumer.consumeWithJsonb(uriInfo.getBaseUri().toString()
        + "artists");

      for (int i = 0; i < artists.length; i++) {
        if (artists[i].name.equals(artist)) {
          return artists[i].albums.length;
        }
      }
      return -1;
    }

    @GET
    @Path("total")
    @Produces(MediaType.TEXT_PLAIN)
    public int getTotalArtists() {
      return Consumer.consumeWithJsonp(uriInfo.getBaseUri().toString() +
        "artists").length;
    }
}
```
{: codeblock}



* The **getArtists()** method provides the raw JSON data service that you accessed at the
beginning of this guide.

* The **getJsonString()** method uses JSON-B to return the JSON as a string that will
be used later for testing.

* The **getTotalAlbums()** method uses JSON-B to return the total number of albums present
in the JSON for a particular artist. The method returns -1 if this artist does not exist.

* The **getTotalArtists()** method uses JSON-P to return the total number of artists
present in the JSON.

The methods that you wrote in the **Consumer** class could be written directly in the
**ArtistResource** class. However, if you are consuming a REST resource from a third
party service, you should separate your **GET**/**POST** requests from your data consumption.


# Running the application

The Open Liberty server was started in development mode at the beginning of the guide and all the changes were automatically picked up.

You can find your service at 
```
curl http://localhost:9080/artists
```
{: codeblock}

.

Now, you can access the endpoint at 
```
curl http://localhost:9080/artists/total
```
{: codeblock}

 to see the total number of artists,
and you can access the endpoint at **\http://localhost:9080/artists/total/<artist>** to see a particular artist's total number of albums. 


# Testing deserialization
ConsumingRestIT.java

Create the `ConsumingRestIT` class.
```
touch src/test/java/it/io/openliberty/guides/consumingrest/ConsumingRestIT.java 
```
{: codeblock}


> [File -> Open]guide-rest-client-java/start/src/test/java/it/io/openliberty/guides/consumingrest/ConsumingRestIT.java 
```

package it.io.openliberty.guides.consumingrest;

import static org.junit.jupiter.api.Assertions.assertEquals;

import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.Response;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;

import io.openliberty.guides.consumingrest.model.Artist;

public class ConsumingRestIT {

    private static String port;
    private static String baseUrl;
    private static String targetUrl;

    private Client client;
    private Response response;

    @BeforeAll
    public static void oneTimeSetup() {
      port = System.getProperty("http.port");
      baseUrl = "http://localhost:" + port + "/artists/";
      targetUrl = baseUrl + "total/";
    }

    @BeforeEach
    public void setup() {
      client = ClientBuilder.newClient();
    }

    @AfterEach
    public void teardown() {
      client.close();
    }

    @Test
    public void testArtistDeserialization() {
      response = client.target(baseUrl + "jsonString").request().get();
      this.assertResponse(baseUrl + "jsonString", response);

      Jsonb jsonb = JsonbBuilder.create();

      String expectedString = "{\"name\":\"foo\",\"albums\":"
        + "[{\"title\":\"album_one\",\"artist\":\"foo\",\"ntracks\":12}]}";
      Artist expected = jsonb.fromJson(expectedString, Artist.class);

      String actualString = response.readEntity(String.class);
		  Artist[] actual = jsonb.fromJson(actualString, Artist[].class);

      assertEquals(expected.name, actual[0].name, 
        "Expected names of artists does not match");

      response.close();
    }

    @Test
    public void testJsonBAlbumCount() {
      String[] artists = {"dj", "bar", "foo"};
      for (int i = 0; i < artists.length; i++) {
        response = client.target(targetUrl + artists[i]).request().get();
        this.assertResponse(targetUrl + artists[i], response);

        int expected = i;
        int actual = response.readEntity(int.class);
        assertEquals(expected, actual, "Album count for " + artists[i] + " does not match");

        response.close();
      }
    }

    @Test
    public void testJsonBAlbumCountForUnknownArtist() {
      response = client.target(targetUrl + "unknown-artist").request().get();

      int expected = -1;
      int actual = response.readEntity(int.class);
      assertEquals(expected, actual, "Unknown artist must have -1 albums");

      response.close();
    }

    @Test
    public void testJsonPArtistCount() {
      response = client.target(targetUrl).request().get();
      this.assertResponse(targetUrl, response);

      int expected = 3;
      int actual = response.readEntity(int.class);
      assertEquals(expected, actual, "Expected number of artists does not match");

      response.close();
    }

     * Asserts that the given URL has the correct (200) response code.
     */
    private void assertResponse(String url, Response response) {
      assertEquals(200, response.getStatus(), "Incorrect response code from " + url);
    }
}
```
{: codeblock}



Maven finds and executes all tests under the **src/test/java/it/** directory, 
and each test method must be marked with the **@Test** annotation.

You can use the **@BeforeAll** and **@AfterAll** annotations to perform any one-time setup and teardown
tasks before and after all of your tests run. You can also use the **@BeforeEach** and **@AfterEach** annotations
to perform setup and teardown tasks for individual test cases.

### Testing the binding process
ConsumingRestIT.java

pom.xml

The **yasson** dependency was added in your **pom.xml** file so that your test classes have access to JSON-B.

The **testArtistDeserialization** test case checks that **Artist** instances created from
the REST data and those that are hardcoded perform the same.

The **assertResponse** helper method ensures that the response code you receive is valid (200).

### Processing with JSON-B test
ConsumingRestIT.java

The **testJsonBAlbumCount** and **testJsonBAlbumCountForUnknownArtist** tests both use the **total/{artist}**
endpoint which invokes JSON-B.

The **testJsonBAlbumCount** test case checks that deserialization with JSON-B was done correctly
and that the correct number of albums is returned for each artist in the JSON.

The **testJsonBAlbumCountForUnknownArtist** test case is similar to **testJsonBAlbumCount**
but instead checks an artist that does not exist in the JSON and ensures that a
value of **-1** is returned.

### Processing with JSON-P test
ConsumingRestIT.java

The **testJsonPArtistCount** test uses the **total** endpoint which invokes JSON-P. This test
checks that deserialization with JSON-P was done correctly and that the correct number
of artists is returned.


### Running the tests

Since you started Open Liberty in development mode at the start of the guide, press the **enter/return** key to run the tests.

If the tests pass, you see a similar output to the following example:

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running it.io.openliberty.guides.consumingrest.ConsumingRestIT
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.59 sec - in it.io.openliberty.guides.consumingrest.ConsumingRestIT

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0


When you are done checking out the service, exit development mode by typing **q** in the command-line session where you ran the server, 
and then press the **enter/return** key.

# Building the application

If you are satisfied with your application, run the Maven **package** goal to build the WAR file in the **target** directory:

mvn package

# Summary

## Clean up your environment

Delete the **guide-rest-client-java** project by navigating to the **/home/project/** directory

```
cd ../..
rm -r -f guide-rest-client-java
rmdir guide-rest-client-java
```
{: codeblock}


# Great work! You're done!

You just accessed a simple RESTful web service and consumed its resources by using JSON-B and JSON-P in Open Liberty.



# Consuming a RESTful web service
## What you'll learn


You will learn how to access a REST service, serialize a Java object that contains a
list of artists and their albums, and use two different approaches to deserialize
the returned JSON resources. The first approach consists of using the Java API for JSON Binding (JSON-B)
to directly convert JSON messages into Java objects. The second approach consists of
using the Java API for JSON Processing (JSON-P) to process the JSON.

The REST service that provides the artists and albums resources has already been written
for you and is accessible at the following link when the server is running 
```
curl http://localhost:9080/artists
```
{: codeblock}

, which responds with the [hotspot]`artists.json`.

You will implement the following two endpoints using the two deserialization approaches:

* **.../artists/total** to return the total number of artists in the JSON
* **.../artists/total/<artist>** to return the total number of albums in the JSON
for the particular artist

If you are interested in learning more about REST services and how you can write them, read
[Creating a RESTful web service](https://openliberty.io/guides/rest-intro.html).



# Getting Started

If a terminal window does not open navigate:

> Terminal -> New Terminal

Check you are in the **home/project** folder:

```
pwd
```
{: codeblock}

The fastest way to work through this guide is to clone the Git repository and use the projects that are provided inside:

```
git clone https://github.com/openliberty/guide-rest-client-java.git
cd guide-rest-client-java
```
{: codeblock}

The **start** directory contains the starting project that you will build upon.



## Try what you’ll build
The **finish** directory in the root of this guide contains the finished application. Give it a try before you proceed.

To try out the application, first navigate to the finish directory and then run the following Maven goal to build the application and run it inside Open Liberty:
```
cd finish
mvn install liberty:start-server
```
{: codeblock}



You can find your service at http://localhost:9080/artists.

Now, you can access the endpoint at http://localhost:9080/artists/total to see the total number of artists, and you can access the endpoint at **\http://localhost:9080/artists/total/<artist>** to see a particular artist`s total number of albums.

After you are finished checking out the application, stop the Open Liberty server by pressing `CTRL+C` in the shell session where you ran the server. Alternatively, you can run the `liberty:stop` goal from the `finish` directory in another shell session:

```
mvn liberty:stop
```
{: codeblock}
After you are done checking out the application, stop the Open Liberty server:

```
mvn liberty:stop-server
```
{: codeblock}


# Starting the service

Navigate to the **start** directory to begin.

```
cd start
```
{: codeblock}



This guide is already setup with a general application. As you progress through the guide,
you will update the code directly and automatically see the results by using development mode.

Start Open Liberty in development mode, which starts the Open Liberty server and listens for file changes:

mvn liberty:dev

When the server is running, you can find your service at the 
```
curl http://localhost:9080/artists
```
{: codeblock}

 URL.

# Creating POJOs





To deserialize a JSON message, start with creating Plain Old Java Objects (POJOs) that represent what
is in the JSON and whose instance members map to the keys in the JSON.

For the purpose of this guide, you are given two POJOs.
The **Artist** object has two instance members **name** and **albums**, 
which map to the artist name and the collection of the albums they have written. The **Album** object represents a 
single object within the album collection, and contains three instance members **title**, **artistName**, and **totalTracks**, which map to the album title, the artist who wrote the album, and the number of tracks the album contains.

# Introducing JSON-B and JSON-P

JSON-B is a feature introduced with Java EE 8 and strengthens Java support for JSON.
With JSON-B you directly serialize and deserialize POJOs. This API gives you a
variety of options for working with JSON resources.

In contrast, you need to use helper methods with JSON-P to process a JSON response.
This tactic is more straightforward, but it can be cumbersome with more complex classes.

JSON-B is built on top of the existing JSON-P API. JSON-B can do everything that JSON-P can do
and allows for more customization for serializing and deserializing.

### Using JSON-B


JSON-B requires a POJO to have a public default no-argument constructor for deserialization
and binding to work properly.

The JSON-B engine includes a set of default mapping rules, which can be run without
any customization annotations or custom configuration. In some instances, you might
find it useful to deserialize a JSON message with only certain fields, specific
field names, or classes with custom constructors. In these cases,
annotations are necessary and recommended:

* The **@JsonbProperty** annotation to map JSON keys to class instance members and vice versa.
Without the use of this annotation, JSON-B will attempt to do POJO mapping, matching the keys in
the JSON to the class instance members by name. JSON-B will attempt to match the JSON key 
with a Java field or method annotated with **@JsonbProperty** where the value in the
annotation exactly matches the JSON key. If no annotation exists with the given JSON key, 
JSON-B will attempt to find a matching field with the same name. If no match is found, 
JSON-B attempts to find a matching getter method for serialization or a matching setter 
method for de-serialization. A match occurs when the property name of the method matches 
the JSON key. If no matching getter or setter method is found, serialization or 
de-serialization, respectively, fails with an exception. The Artist POJO does not require 
this annotation because all instance members match the JSON keys by name.

* The **@JsonbCreator** and **@JsonbProperty** annotations to annotate a custom constructor.
These annotations are required for proper parameter substitution when a custom constructor is used.

* The **@JsonbTransient** annotation to define an object property that does not map to a JSON
property. While the use of this annotation is good practice, it is only necessary for serialization.

For more information on customization with JSON-B, see the [official JSON-B site](http://json-b.net).


# Consuming the REST resource







The **Artist** and **Album** POJOs are ready for deserialization. 
Next, we'll learn to consume the JSON response from your REST service.

Create the `Consumer` class.
```
touch src/main/java/io/openliberty/guides/consumingrest/Consumer.java
```
{: codeblock}


> [File -> Open]guide-rest-client-java/start/src/main/java/io/openliberty/guides/consumingrest/Consumer.java
```

package io.openliberty.guides.consumingrest;

import java.util.List;
import java.util.stream.Collectors;

import javax.json.JsonArray;
import javax.json.JsonObject;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.Response;

import io.openliberty.guides.consumingrest.model.Album;
import io.openliberty.guides.consumingrest.model.Artist;

public class Consumer {
    public static Artist[] consumeWithJsonb(String targetUrl) {
      Client client = ClientBuilder.newClient();
      Response response = client.target(targetUrl).request().get();
      Artist[] artists = response.readEntity(Artist[].class);

      response.close();
      client.close();

      return artists;
    }

    public static Artist[] consumeWithJsonp(String targetUrl) {
      Client client = ClientBuilder.newClient();
      Response response = client.target(targetUrl).request().get();
      JsonArray arr = response.readEntity(JsonArray.class);

      response.close();
      client.close();

      return Consumer.collectArtists(arr);
    }

    private static Artist[] collectArtists(JsonArray artistArr) {
      List<Artist> artists = artistArr.stream().map(artistJson -> {
        JsonArray albumArr = ((JsonObject) artistJson).getJsonArray("albums");
        Artist artist = new Artist(
          ((JsonObject) artistJson).getString("name"),
          Consumer.collectAlbums(albumArr));
        return artist;
      }).collect(Collectors.toList());

      return artists.toArray(new Artist[artists.size()]);
    }

    private static Album[] collectAlbums(JsonArray albumArr) {
      List<Album> albums = albumArr.stream().map(albumJson -> {
        Album album = new Album(
          ((JsonObject) albumJson).getString("title"),
          ((JsonObject) albumJson).getString("artist"),
          ((JsonObject) albumJson).getInt("ntracks") );
        return album;
      }).collect(Collectors.toList());

      return albums.toArray(new Album[albums.size()]);
    }
}
```
{: codeblock}



### Processing JSON using JSON-B




JSON-B is a Java API that is used to serialize Java objects to JSON messages and vice versa.

Open Liberty's JSON-B feature on Maven Central includes the JSON-B provider through transitive dependencies.
The JSON-B APIs are provided by the MicroProfile dependency in your **pom.xml** file. 
Look for the dependency with the **microprofile** artifact ID. 

The **consumeWithJsonb()** method in the **Consumer** class makes a **GET** request to the
running artist service and retrieves the JSON. To bind the JSON into an **Artist**
array, use the **Artist[]** entity type in the **readEntity** call.

### Processing JSON using JSON-P


The **consumeWithJsonp()** method in the **Consumer** class makes a **GET** request
to the running artist service and retrieves the JSON. This method then uses the
**collectArtists** and **collectAlbums** helper methods. These helper methods will
parse the JSON and collect its objects into individual POJOs. Notice that you can
use the custom constructors to create instances of **Artist** and **Album**.

# Creating additional REST resources




Now that you can consume a JSON resource you can put that data to use.

Replace the `ArtistResource` class.

> [File -> Open]guide-rest-client-java/start/src/main/java/io/openliberty/guides/consumingrest/service/ArtistResource.java



```
package io.openliberty.guides.consumingrest.service;

import javax.json.JsonArray;
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.UriInfo;

import io.openliberty.guides.consumingrest.model.Artist;
import io.openliberty.guides.consumingrest.Consumer;

@Path("artists")
public class ArtistResource {

    @Context
    UriInfo uriInfo;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public JsonArray getArtists() {
    	return Reader.getArtists();
    }

    @GET
    @Path("jsonString")
    @Produces(MediaType.TEXT_PLAIN)
    public String getJsonString() {
      Jsonb jsonb = JsonbBuilder.create();

      Artist[] artists = Consumer.consumeWithJsonb(uriInfo.getBaseUri().toString() +
        "artists");
      String result = jsonb.toJson(artists);

      return result;
    }

    @GET
    @Path("total/{artist}")
    @Produces(MediaType.TEXT_PLAIN)
    public int getTotalAlbums(@PathParam("artist") String artist) {
      Artist[] artists = Consumer.consumeWithJsonb(uriInfo.getBaseUri().toString()
        + "artists");

      for (int i = 0; i < artists.length; i++) {
        if (artists[i].name.equals(artist)) {
          return artists[i].albums.length;
        }
      }
      return -1;
    }

    @GET
    @Path("total")
    @Produces(MediaType.TEXT_PLAIN)
    public int getTotalArtists() {
      return Consumer.consumeWithJsonp(uriInfo.getBaseUri().toString() +
        "artists").length;
    }
}
```
{: codeblock}



* The **getArtists()** method provides the raw JSON data service that you accessed at the
beginning of this guide.

* The **getJsonString()** method uses JSON-B to return the JSON as a string that will
be used later for testing.

* The **getTotalAlbums()** method uses JSON-B to return the total number of albums present
in the JSON for a particular artist. The method returns -1 if this artist does not exist.

* The **getTotalArtists()** method uses JSON-P to return the total number of artists
present in the JSON.

The methods that you wrote in the **Consumer** class could be written directly in the
**ArtistResource** class. However, if you are consuming a REST resource from a third
party service, you should separate your **GET**/**POST** requests from your data consumption.


# Running the application

The Open Liberty server was started in development mode at the beginning of the guide and all the changes were automatically picked up.

You can find your service at 
```
curl http://localhost:9080/artists
```
{: codeblock}

.

Now, you can access the endpoint at 
```
curl http://localhost:9080/artists/total
```
{: codeblock}

 to see the total number of artists,
and you can access the endpoint at **\http://localhost:9080/artists/total/<artist>** to see a particular artist's total number of albums. 


# Testing deserialization


Create the `ConsumingRestIT` class.
```
touch src/test/java/it/io/openliberty/guides/consumingrest/ConsumingRestIT.java 
```
{: codeblock}


> [File -> Open]guide-rest-client-java/start/src/test/java/it/io/openliberty/guides/consumingrest/ConsumingRestIT.java 
```

package it.io.openliberty.guides.consumingrest;

import static org.junit.jupiter.api.Assertions.assertEquals;

import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.Response;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;

import io.openliberty.guides.consumingrest.model.Artist;

public class ConsumingRestIT {

    private static String port;
    private static String baseUrl;
    private static String targetUrl;

    private Client client;
    private Response response;

    @BeforeAll
    public static void oneTimeSetup() {
      port = System.getProperty("http.port");
      baseUrl = "http://localhost:" + port + "/artists/";
      targetUrl = baseUrl + "total/";
    }

    @BeforeEach
    public void setup() {
      client = ClientBuilder.newClient();
    }

    @AfterEach
    public void teardown() {
      client.close();
    }

    @Test
    public void testArtistDeserialization() {
      response = client.target(baseUrl + "jsonString").request().get();
      this.assertResponse(baseUrl + "jsonString", response);

      Jsonb jsonb = JsonbBuilder.create();

      String expectedString = "{\"name\":\"foo\",\"albums\":"
        + "[{\"title\":\"album_one\",\"artist\":\"foo\",\"ntracks\":12}]}";
      Artist expected = jsonb.fromJson(expectedString, Artist.class);

      String actualString = response.readEntity(String.class);
		  Artist[] actual = jsonb.fromJson(actualString, Artist[].class);

      assertEquals(expected.name, actual[0].name, 
        "Expected names of artists does not match");

      response.close();
    }

    @Test
    public void testJsonBAlbumCount() {
      String[] artists = {"dj", "bar", "foo"};
      for (int i = 0; i < artists.length; i++) {
        response = client.target(targetUrl + artists[i]).request().get();
        this.assertResponse(targetUrl + artists[i], response);

        int expected = i;
        int actual = response.readEntity(int.class);
        assertEquals(expected, actual, "Album count for " + artists[i] + " does not match");

        response.close();
      }
    }

    @Test
    public void testJsonBAlbumCountForUnknownArtist() {
      response = client.target(targetUrl + "unknown-artist").request().get();

      int expected = -1;
      int actual = response.readEntity(int.class);
      assertEquals(expected, actual, "Unknown artist must have -1 albums");

      response.close();
    }

    @Test
    public void testJsonPArtistCount() {
      response = client.target(targetUrl).request().get();
      this.assertResponse(targetUrl, response);

      int expected = 3;
      int actual = response.readEntity(int.class);
      assertEquals(expected, actual, "Expected number of artists does not match");

      response.close();
    }

     * Asserts that the given URL has the correct (200) response code.
     */
    private void assertResponse(String url, Response response) {
      assertEquals(200, response.getStatus(), "Incorrect response code from " + url);
    }
}
```
{: codeblock}



Maven finds and executes all tests under the **src/test/java/it/** directory, 
and each test method must be marked with the **@Test** annotation.

You can use the **@BeforeAll** and **@AfterAll** annotations to perform any one-time setup and teardown
tasks before and after all of your tests run. You can also use the **@BeforeEach** and **@AfterEach** annotations
to perform setup and teardown tasks for individual test cases.

### Testing the binding process




The **yasson** dependency was added in your **pom.xml** file so that your test classes have access to JSON-B.

The **testArtistDeserialization** test case checks that **Artist** instances created from
the REST data and those that are hardcoded perform the same.

The **assertResponse** helper method ensures that the response code you receive is valid (200).

### Processing with JSON-B test


The **testJsonBAlbumCount** and **testJsonBAlbumCountForUnknownArtist** tests both use the **total/{artist}**
endpoint which invokes JSON-B.

The **testJsonBAlbumCount** test case checks that deserialization with JSON-B was done correctly
and that the correct number of albums is returned for each artist in the JSON.

The **testJsonBAlbumCountForUnknownArtist** test case is similar to **testJsonBAlbumCount**
but instead checks an artist that does not exist in the JSON and ensures that a
value of **-1** is returned.

### Processing with JSON-P test


The **testJsonPArtistCount** test uses the **total** endpoint which invokes JSON-P. This test
checks that deserialization with JSON-P was done correctly and that the correct number
of artists is returned.


### Running the tests

Since you started Open Liberty in development mode at the start of the guide, press the **enter/return** key to run the tests.

If the tests pass, you see a similar output to the following example:

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running it.io.openliberty.guides.consumingrest.ConsumingRestIT
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.59 sec - in it.io.openliberty.guides.consumingrest.ConsumingRestIT

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0


When you are done checking out the service, exit development mode by typing **q** in the command-line session where you ran the server, 
and then press the **enter/return** key.

# Building the application

If you are satisfied with your application, run the Maven **package** goal to build the WAR file in the **target** directory:

mvn package

# Summary

## Clean up your environment

Delete the **guide-rest-client-java** project by navigating to the **/home/project/** directory

```
cd ../..
rm -r -f guide-rest-client-java
rmdir guide-rest-client-java
```
{: codeblock}


# Great work! You're done!

You just accessed a simple RESTful web service and consumed its resources by using JSON-B and JSON-P in Open Liberty.



# Consuming a RESTful web service
## What you'll learn


You will learn how to access a REST service, serialize a Java object that contains a
list of artists and their albums, and use two different approaches to deserialize
the returned JSON resources. The first approach consists of using the Java API for JSON Binding (JSON-B)
to directly convert JSON messages into Java objects. The second approach consists of
using the Java API for JSON Processing (JSON-P) to process the JSON.

The REST service that provides the artists and albums resources has already been written
for you and is accessible at the following link when the server is running 
```
curl http://localhost:9080/artists
```
{: codeblock}

, which responds with the [hotspot]`artists.json`.

You will implement the following two endpoints using the two deserialization approaches:

* **.../artists/total** to return the total number of artists in the JSON
* **.../artists/total/<artist>** to return the total number of albums in the JSON
for the particular artist

If you are interested in learning more about REST services and how you can write them, read
[Creating a RESTful web service](https://openliberty.io/guides/rest-intro.html).



# Getting Started

If a terminal window does not open navigate:

> Terminal -> New Terminal

Check you are in the **home/project** folder:

```
pwd
```
{: codeblock}

The fastest way to work through this guide is to clone the Git repository and use the projects that are provided inside:

```
git clone https://github.com/openliberty/guide-rest-client-java.git
cd guide-rest-client-java
```
{: codeblock}

The **start** directory contains the starting project that you will build upon.



## Try what you’ll build
The **finish** directory in the root of this guide contains the finished application. Give it a try before you proceed.

To try out the application, first navigate to the finish directory and then run the following Maven goal to build the application and run it inside Open Liberty:
```
cd finish
mvn install liberty:start-server
```
{: codeblock}



You can find your service at http://localhost:9080/artists.

Now, you can access the endpoint at http://localhost:9080/artists/total to see the total number of artists, and you can access the endpoint at **\http://localhost:9080/artists/total/<artist>** to see a particular artist`s total number of albums.

After you are finished checking out the application, stop the Open Liberty server by pressing `CTRL+C` in the shell session where you ran the server. Alternatively, you can run the `liberty:stop` goal from the `finish` directory in another shell session:

```
mvn liberty:stop
```
{: codeblock}
After you are done checking out the application, stop the Open Liberty server:

```
mvn liberty:stop-server
```
{: codeblock}


# Starting the service

Navigate to the **start** directory to begin.

```
cd start
```
{: codeblock}



This guide is already setup with a general application. As you progress through the guide,
you will update the code directly and automatically see the results by using development mode.

Start Open Liberty in development mode, which starts the Open Liberty server and listens for file changes:

mvn liberty:dev

When the server is running, you can find your service at the 
```
curl http://localhost:9080/artists
```
{: codeblock}

 URL.

# Creating POJOs





To deserialize a JSON message, start with creating Plain Old Java Objects (POJOs) that represent what
is in the JSON and whose instance members map to the keys in the JSON.

For the purpose of this guide, you are given two POJOs.
The **Artist** object has two instance members **name** and **albums**, 
which map to the artist name and the collection of the albums they have written. The **Album** object represents a 
single object within the album collection, and contains three instance members **title**, **artistName**, and **totalTracks**, which map to the album title, the artist who wrote the album, and the number of tracks the album contains.

# Introducing JSON-B and JSON-P

JSON-B is a feature introduced with Java EE 8 and strengthens Java support for JSON.
With JSON-B you directly serialize and deserialize POJOs. This API gives you a
variety of options for working with JSON resources.

In contrast, you need to use helper methods with JSON-P to process a JSON response.
This tactic is more straightforward, but it can be cumbersome with more complex classes.

JSON-B is built on top of the existing JSON-P API. JSON-B can do everything that JSON-P can do
and allows for more customization for serializing and deserializing.

### Using JSON-B


JSON-B requires a POJO to have a public default no-argument constructor for deserialization
and binding to work properly.

The JSON-B engine includes a set of default mapping rules, which can be run without
any customization annotations or custom configuration. In some instances, you might
find it useful to deserialize a JSON message with only certain fields, specific
field names, or classes with custom constructors. In these cases,
annotations are necessary and recommended:

* The **@JsonbProperty** annotation to map JSON keys to class instance members and vice versa.
Without the use of this annotation, JSON-B will attempt to do POJO mapping, matching the keys in
the JSON to the class instance members by name. JSON-B will attempt to match the JSON key 
with a Java field or method annotated with **@JsonbProperty** where the value in the
annotation exactly matches the JSON key. If no annotation exists with the given JSON key, 
JSON-B will attempt to find a matching field with the same name. If no match is found, 
JSON-B attempts to find a matching getter method for serialization or a matching setter 
method for de-serialization. A match occurs when the property name of the method matches 
the JSON key. If no matching getter or setter method is found, serialization or 
de-serialization, respectively, fails with an exception. The Artist POJO does not require 
this annotation because all instance members match the JSON keys by name.

* The **@JsonbCreator** and **@JsonbProperty** annotations to annotate a custom constructor.
These annotations are required for proper parameter substitution when a custom constructor is used.

* The **@JsonbTransient** annotation to define an object property that does not map to a JSON
property. While the use of this annotation is good practice, it is only necessary for serialization.

For more information on customization with JSON-B, see the [official JSON-B site](http://json-b.net).


# Consuming the REST resource







The **Artist** and **Album** POJOs are ready for deserialization. 
Next, we'll learn to consume the JSON response from your REST service.

Create the `Consumer` class.
```
touch src/main/java/io/openliberty/guides/consumingrest/Consumer.java
```
{: codeblock}


> [File -> Open]guide-rest-client-java/start/src/main/java/io/openliberty/guides/consumingrest/Consumer.java
```

package io.openliberty.guides.consumingrest;

import java.util.List;
import java.util.stream.Collectors;

import javax.json.JsonArray;
import javax.json.JsonObject;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.Response;

import io.openliberty.guides.consumingrest.model.Album;
import io.openliberty.guides.consumingrest.model.Artist;

public class Consumer {
    public static Artist[] consumeWithJsonb(String targetUrl) {
      Client client = ClientBuilder.newClient();
      Response response = client.target(targetUrl).request().get();
      Artist[] artists = response.readEntity(Artist[].class);

      response.close();
      client.close();

      return artists;
    }

    public static Artist[] consumeWithJsonp(String targetUrl) {
      Client client = ClientBuilder.newClient();
      Response response = client.target(targetUrl).request().get();
      JsonArray arr = response.readEntity(JsonArray.class);

      response.close();
      client.close();

      return Consumer.collectArtists(arr);
    }

    private static Artist[] collectArtists(JsonArray artistArr) {
      List<Artist> artists = artistArr.stream().map(artistJson -> {
        JsonArray albumArr = ((JsonObject) artistJson).getJsonArray("albums");
        Artist artist = new Artist(
          ((JsonObject) artistJson).getString("name"),
          Consumer.collectAlbums(albumArr));
        return artist;
      }).collect(Collectors.toList());

      return artists.toArray(new Artist[artists.size()]);
    }

    private static Album[] collectAlbums(JsonArray albumArr) {
      List<Album> albums = albumArr.stream().map(albumJson -> {
        Album album = new Album(
          ((JsonObject) albumJson).getString("title"),
          ((JsonObject) albumJson).getString("artist"),
          ((JsonObject) albumJson).getInt("ntracks") );
        return album;
      }).collect(Collectors.toList());

      return albums.toArray(new Album[albums.size()]);
    }
}
```
{: codeblock}



### Processing JSON using JSON-B




JSON-B is a Java API that is used to serialize Java objects to JSON messages and vice versa.

Open Liberty's JSON-B feature on Maven Central includes the JSON-B provider through transitive dependencies.
The JSON-B APIs are provided by the MicroProfile dependency in your **pom.xml** file. 
Look for the dependency with the **microprofile** artifact ID. 

The **consumeWithJsonb()** method in the **Consumer** class makes a **GET** request to the
running artist service and retrieves the JSON. To bind the JSON into an **Artist**
array, use the **Artist[]** entity type in the **readEntity** call.

### Processing JSON using JSON-P


The **consumeWithJsonp()** method in the **Consumer** class makes a **GET** request
to the running artist service and retrieves the JSON. This method then uses the
**collectArtists** and **collectAlbums** helper methods. These helper methods will
parse the JSON and collect its objects into individual POJOs. Notice that you can
use the custom constructors to create instances of **Artist** and **Album**.

# Creating additional REST resources




Now that you can consume a JSON resource you can put that data to use.

Replace the `ArtistResource` class.

> [File -> Open]guide-rest-client-java/start/src/main/java/io/openliberty/guides/consumingrest/service/ArtistResource.java



```
package io.openliberty.guides.consumingrest.service;

import javax.json.JsonArray;
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.UriInfo;

import io.openliberty.guides.consumingrest.model.Artist;
import io.openliberty.guides.consumingrest.Consumer;

@Path("artists")
public class ArtistResource {

    @Context
    UriInfo uriInfo;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public JsonArray getArtists() {
    	return Reader.getArtists();
    }

    @GET
    @Path("jsonString")
    @Produces(MediaType.TEXT_PLAIN)
    public String getJsonString() {
      Jsonb jsonb = JsonbBuilder.create();

      Artist[] artists = Consumer.consumeWithJsonb(uriInfo.getBaseUri().toString() +
        "artists");
      String result = jsonb.toJson(artists);

      return result;
    }

    @GET
    @Path("total/{artist}")
    @Produces(MediaType.TEXT_PLAIN)
    public int getTotalAlbums(@PathParam("artist") String artist) {
      Artist[] artists = Consumer.consumeWithJsonb(uriInfo.getBaseUri().toString()
        + "artists");

      for (int i = 0; i < artists.length; i++) {
        if (artists[i].name.equals(artist)) {
          return artists[i].albums.length;
        }
      }
      return -1;
    }

    @GET
    @Path("total")
    @Produces(MediaType.TEXT_PLAIN)
    public int getTotalArtists() {
      return Consumer.consumeWithJsonp(uriInfo.getBaseUri().toString() +
        "artists").length;
    }
}
```
{: codeblock}



* The **getArtists()** method provides the raw JSON data service that you accessed at the
beginning of this guide.

* The **getJsonString()** method uses JSON-B to return the JSON as a string that will
be used later for testing.

* The **getTotalAlbums()** method uses JSON-B to return the total number of albums present
in the JSON for a particular artist. The method returns -1 if this artist does not exist.

* The **getTotalArtists()** method uses JSON-P to return the total number of artists
present in the JSON.

The methods that you wrote in the **Consumer** class could be written directly in the
**ArtistResource** class. However, if you are consuming a REST resource from a third
party service, you should separate your **GET**/**POST** requests from your data consumption.


# Running the application

The Open Liberty server was started in development mode at the beginning of the guide and all the changes were automatically picked up.

You can find your service at 
```
curl http://localhost:9080/artists
```
{: codeblock}

.

Now, you can access the endpoint at 
```
curl http://localhost:9080/artists/total
```
{: codeblock}

 to see the total number of artists,
and you can access the endpoint at **\http://localhost:9080/artists/total/<artist>** to see a particular artist's total number of albums. 


# Testing deserialization


Create the `ConsumingRestIT` class.
```
touch src/test/java/it/io/openliberty/guides/consumingrest/ConsumingRestIT.java 
```
{: codeblock}


> [File -> Open]guide-rest-client-java/start/src/test/java/it/io/openliberty/guides/consumingrest/ConsumingRestIT.java 
```

package it.io.openliberty.guides.consumingrest;

import static org.junit.jupiter.api.Assertions.assertEquals;

import javax.json.bind.Jsonb;
import javax.json.bind.JsonbBuilder;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.Response;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;

import io.openliberty.guides.consumingrest.model.Artist;

public class ConsumingRestIT {

    private static String port;
    private static String baseUrl;
    private static String targetUrl;

    private Client client;
    private Response response;

    @BeforeAll
    public static void oneTimeSetup() {
      port = System.getProperty("http.port");
      baseUrl = "http://localhost:" + port + "/artists/";
      targetUrl = baseUrl + "total/";
    }

    @BeforeEach
    public void setup() {
      client = ClientBuilder.newClient();
    }

    @AfterEach
    public void teardown() {
      client.close();
    }

    @Test
    public void testArtistDeserialization() {
      response = client.target(baseUrl + "jsonString").request().get();
      this.assertResponse(baseUrl + "jsonString", response);

      Jsonb jsonb = JsonbBuilder.create();

      String expectedString = "{\"name\":\"foo\",\"albums\":"
        + "[{\"title\":\"album_one\",\"artist\":\"foo\",\"ntracks\":12}]}";
      Artist expected = jsonb.fromJson(expectedString, Artist.class);

      String actualString = response.readEntity(String.class);
		  Artist[] actual = jsonb.fromJson(actualString, Artist[].class);

      assertEquals(expected.name, actual[0].name, 
        "Expected names of artists does not match");

      response.close();
    }

    @Test
    public void testJsonBAlbumCount() {
      String[] artists = {"dj", "bar", "foo"};
      for (int i = 0; i < artists.length; i++) {
        response = client.target(targetUrl + artists[i]).request().get();
        this.assertResponse(targetUrl + artists[i], response);

        int expected = i;
        int actual = response.readEntity(int.class);
        assertEquals(expected, actual, "Album count for " + artists[i] + " does not match");

        response.close();
      }
    }

    @Test
    public void testJsonBAlbumCountForUnknownArtist() {
      response = client.target(targetUrl + "unknown-artist").request().get();

      int expected = -1;
      int actual = response.readEntity(int.class);
      assertEquals(expected, actual, "Unknown artist must have -1 albums");

      response.close();
    }

    @Test
    public void testJsonPArtistCount() {
      response = client.target(targetUrl).request().get();
      this.assertResponse(targetUrl, response);

      int expected = 3;
      int actual = response.readEntity(int.class);
      assertEquals(expected, actual, "Expected number of artists does not match");

      response.close();
    }

     * Asserts that the given URL has the correct (200) response code.
     */
    private void assertResponse(String url, Response response) {
      assertEquals(200, response.getStatus(), "Incorrect response code from " + url);
    }
}
```
{: codeblock}



Maven finds and executes all tests under the **src/test/java/it/** directory, 
and each test method must be marked with the **@Test** annotation.

You can use the **@BeforeAll** and **@AfterAll** annotations to perform any one-time setup and teardown
tasks before and after all of your tests run. You can also use the **@BeforeEach** and **@AfterEach** annotations
to perform setup and teardown tasks for individual test cases.

### Testing the binding process




The **yasson** dependency was added in your **pom.xml** file so that your test classes have access to JSON-B.

The **testArtistDeserialization** test case checks that **Artist** instances created from
the REST data and those that are hardcoded perform the same.

The **assertResponse** helper method ensures that the response code you receive is valid (200).

### Processing with JSON-B test


The **testJsonBAlbumCount** and **testJsonBAlbumCountForUnknownArtist** tests both use the **total/{artist}**
endpoint which invokes JSON-B.

The **testJsonBAlbumCount** test case checks that deserialization with JSON-B was done correctly
and that the correct number of albums is returned for each artist in the JSON.

The **testJsonBAlbumCountForUnknownArtist** test case is similar to **testJsonBAlbumCount**
but instead checks an artist that does not exist in the JSON and ensures that a
value of **-1** is returned.

### Processing with JSON-P test


The **testJsonPArtistCount** test uses the **total** endpoint which invokes JSON-P. This test
checks that deserialization with JSON-P was done correctly and that the correct number
of artists is returned.


### Running the tests

Since you started Open Liberty in development mode at the start of the guide, press the **enter/return** key to run the tests.

If the tests pass, you see a similar output to the following example:

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running it.io.openliberty.guides.consumingrest.ConsumingRestIT
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.59 sec - in it.io.openliberty.guides.consumingrest.ConsumingRestIT

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0


When you are done checking out the service, exit development mode by typing **q** in the command-line session where you ran the server, 
and then press the **enter/return** key.

# Building the application

If you are satisfied with your application, run the Maven **package** goal to build the WAR file in the **target** directory:

mvn package

# Summary

## Clean up your environment

Delete the **guide-rest-client-java** project by navigating to the **/home/project/** directory

```
cd ../..
rm -r -f guide-rest-client-java
rmdir guide-rest-client-java
```
{: codeblock}


# Great work! You're done!

You just accessed a simple RESTful web service and consumed its resources by using JSON-B and JSON-P in Open Liberty.


