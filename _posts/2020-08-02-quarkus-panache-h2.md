---
layout: post
title:  "Quarkus ORM with Panache and H2"
date:   2020-08-02 14:31:44 -0300
categories: quarkus panache h2
---

This tutorial is based on:
[*Quarkus - Simplified Hibernate ORM WITH PANACHE*](https://quarkus.io/guides/hibernate-orm-panache)

### H2 Database Installation

For this tutorial you actually **don't** need to download H2. But it's
nice to have so you can inspect your database.

  - Download the "Last Stable" version from [H2 website](https://h2database.com/html/main.html)
  - Extract the file
  - Go to h2 directory
  - Run: `java -jar bin/h2-1.4.199.jar` (ps: replace with appropriate version)
  - It will open the H2 Console on the browser
  - Change `JDBC URL` to `jdbc:h2:~/panache-h2`
  - Click in *Connect*

### Let's create our table

{% highlight sql %}
    create table Person (
       id bigint not null,
        birth date,
        name varchar(255),
        status integer,
        primary key (id)
    )
{% endhighlight %}

After you run the command above, disconnect and close H2 Console.

### Creating the Maven project

{% highlight shell %}
mvn io.quarkus:quarkus-maven-plugin:1.7.0.CR1:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=quarkus-panache-h2 \
    -DclassName="org.acme.hibernate.orm.panache.PersonResource" \
    -Dpath="/people" \
    -Dextensions="resteasy-jsonb,hibernate-orm-panache,jdbc-h2"
cd quarkus-panache-h2
{% endhighlight %}

### Edit ```src/main/resources/application.properties```

{% highlight properties %}
# Configuration file
# key = value

# configure your datasource
quarkus.datasource.db-kind = h2
quarkus.datasource.username = sa
# quarkus.datasource.password = 
quarkus.datasource.jdbc.url = jdbc:h2:~/panache-h2
quarkus.hibernate-orm.log.sql=true
{% endhighlight %}

## Let's create our classes

It's just two entities and one resource.

### Status.java

{% highlight java %}
package org.acme.hibernate.orm.panache;

public enum Status {
    Alive,
    Dead,
}
{% endhighlight %}

### Person.java

{% highlight java %}
package org.acme.hibernate.orm.panache;

import java.time.LocalDate;
import java.util.List;
import javax.persistence.Entity;
import io.quarkus.hibernate.orm.panache.PanacheEntity;

@Entity
public class Person extends PanacheEntity {
    public String name;
    public LocalDate birth;
    public Status status;

    public static Person findByName(String name){
        return find("name", name).firstResult();
    }

    public static List<Person> findAlive(){
        return list("status", Status.Alive);
    }

    public static void deleteStefs(){
        delete("name", "Stef");
    }
}
{% endhighlight %}

[PanacheEntity.java] already has the ID field and it extends [PanacheEntityBase.java]

### PersonResource.java

{% highlight java %}
package org.acme.hibernate.orm.panache;

import java.util.List;

import javax.transaction.Transactional;
import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.PUT;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

@Path("/people")
public class PersonResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/plaintext")
    public List<Person> listAll() {
        return Person.listAll();
    }

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Person> listAllJson() {
        return Person.listAll();
    }

    @POST
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    @Transactional
    public Response create(Person person) {
        person.persist();
        
        return Response.ok(person).status(Response.Status.CREATED).build();
    }
}
{% endhighlight %}

### Let's use Swagger so we can test our APIs

  - Add the following dependency to `pom.xml`:
    {% highlight xml %}
    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-smallrye-openapi</artifactId>
    </dependency>
    {% endhighlight %}
    
  - Add swagger-ui to `application.properties`
  
{% highlight properties %}
# Let's use Swagger so we can test our APIs
quarkus.swagger-ui.always-include=true
{% endhighlight %}

    
  - Start Quarkus: mvn quarkus:dev
  - Access: [http://localhost:8080/swagger-ui/](http://localhost:8080/swagger-ui/)
  - Create some people, eg:
  {% highlight javascript %}
{
  "birth": "2020-08-02",
  "name": "Bob",
  "status": "Alive"
}
{% endhighlight %}
  - Call the GET method to see if they were created.

!!! Important: if you pass the ID field to persist:
{% highlight javascript %}
{
  "id": 1,
  "birth": "2020-08-02",
  "name": "Bob",
  "status": "Alive"
}
{% endhighlight %}

... you'll get the error:

<span style="color:red">Caused by: org.hibernate.PersistentObjectException: detached entity passed to persist: org.acme.hibernate.orm.panache.Person</span>

I think the error occurs because the id is set but no `Person.find` was used,
so the entity is not attached. It's just a guess ... =)

### Now let's update our entity

  - Add to `PersonResource.java`
  {% highlight java %}
    @PUT
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    @Transactional
    public Response update(Person person) {
        Person oldPerson = Person.findById(person.id);
        if (oldPerson == null) 
            return Response.ok(false).status(Response.Status.BAD_REQUEST).build();
        oldPerson.name = person.name;
        oldPerson.birth = person.birth;
        oldPerson.status = person.status;
        oldPerson.persist();
        return Response.ok(true).status(Response.Status.OK).build();
    }
    {% endhighlight %}
  - Reload Swagger UI page and execute the PUT method:
    - Now you have to pass the ID!
    - Update Bob's name to John
	  {% highlight javascript %}
	{
	  "id": 1,
	  "birth": "2020-08-02",
	  "name": "John",
	  "status": "Alive"
	}
	{% endhighlight %}

    - Call the GET method again and check if the update worked.
    
### That's it!

And if you didn't notice, you didn't even need to restart Quarkus!

This is thanks to the amazing **hot-reload** capability that Quarkus provides!
 :man_dancing: :cowboy_hat_face: :sunglasses:

[PanacheEntity.java]: https://github.com/quarkusio/quarkus/blob/master/extensions/panache/hibernate-orm-panache/runtime/src/main/java/io/quarkus/hibernate/orm/panache/PanacheEntity.java
[PanacheEntityBase.java]: https://github.com/quarkusio/quarkus/blob/master/extensions/panache/hibernate-orm-panache/runtime/src/main/java/io/quarkus/hibernate/orm/panache/PanacheEntityBase.java
