---
title: "Getting Started with GraphQL"
categories: [spring-boot]
date: 2020-11-05 06:00:00 +1000
modified: 2020-11-05 06:00:00 +1000
author: pratikdas
excerpt: "Getting Started with GraphQL"
image:
  auto: 0086-twelve
---

GraphQL was developed by Facebook in 2012 for their mobile apps. It was open-sourced in 2015 and is now used by many development teams, including some prominent ones like GitHub, Twitter, and Airbnb. Here we will see what is GraphQL and explain its usage with a simple example.

## What is GraphQL

GraphQL is a specification of a query language for APIs. The client or API consumers sends the request in a query language containing the fields it requires and the server returns only the requested fields instead of the complete payload. 
A sample request/response to a graphQL endpoint looks like this:

```
{
  product {
    name
    rating
    price
  }
}

```


In this sample, we send a request for fetching a list of product with attributes name, and price and server returns the response containing only those fields (name and price). 

GraphQL shifts some responsibility to the client for constructing the query containing the fields of its interest. Apart from this the client also does caching. The server is responsible for resolving the query with a resolver component and then fetching the data from an underlying system like a database or a web service.


## Difference with REST
Good API designs are usually driven by consumer needs which can be varied. If we consider an ecommerce site, we might want to show recent purchases of a customer in an order history page and last viewed products in a profile page. 

With REST we will require two different APIs although we are working with the same product data. Alternately we might fetch the entire product data with all its relations everytime even though we only need a part of the data. GraphQL tries to solve these problems of overfetching or underfetching data by 

With GraphQL, we will have a single endpoint on which the consumer can send different queries depending on the data of interest. For our product example, we will have a graphQL endpoint over which a query for fetching name, rating, and price will look like:

```
{
  product {
    name
    rating
    price
  }
}

```

Similarly, a query for fetching the name with related products will be :

```
{
  product {
    name
    product {
      name
    }
  }

} 

```

So instead of the server providing multiple APIs for the different needs of the consumer, the onus is thrown to the consumer to fetch only the data of its interest.

The consumers of the API can use this query language to request for the specific data of their interest instead of the complete payload from the API response. This is done by creating a strongly typed Schema of our API and instructions on how our API can resolve data and client queries.


## GraphQL SDL: Service, Types, and Fields
GraphQL is language agnostic so it has its own query language and a schema definition language. A `Type` is the most basic component of a GraphQL schema and represents a kind of object we can fetch from our service. 

### Type : Scaler and Object Type
We create a GraphQL service by defining types, and then providing functions for each type. Similar to the types in many programming languages , a type can be a scaler like int, string, decimal, etc or an object type formed with a combination of multiple scaler and complex types. An example of types for a GraphQL service that fetches a list of recent purchases looks like this:
```
type Product {
    id: ID!
    title: String!
    description: String!
    category: String
    madeBy: Manufacturer!
}

type Manufacturer {
    id: ID!
    name: String!
    address: String
}

```
Here we have defined object types: `Product` and `Manufacturer`. `Manufacturer` type is composed of scaler types with names: `id`, `name`, and `address`. Similarly, `Product` type is composed of four scaler types with names: `id`, `title`,`description`, `category`, and an object type `Manufacturer`.

### Special Types: Query, Mutation, and Subscription
We need to add root types of the GraphQL schema for adding functionality to the API. GraphQL Schema has three root level types : Query, Mutation, and Subscription. These are special Types and signify the entry point of a GraphQL service. Of these three, only the Query type is mandatory for every GraphQL service. 

The root types determine the shape of the queries and mutations that will be accepted by the server. An example of Query root type for a GraphQL service that fetches a list of recent purchases looks like this:

```
type Query {
    myRecentPurchases(count: Int, customerID: String): [Product]!
}
```

A mutation is represents changes we can make on our object. Our schema with a mutation will look like this:

```
...
...
type Mutation {
    addPurchases(count: Int, customerID: String): [Product]!
}
```
Here, this mutation is used to add purchases.

Subscription is another special type for real-time push-style updates. Subscriptions depend on the use of a publish mechanism to generate the event that notify a subscription that is subscribed to that event. 

```
type Subscription {
  newProduct: Product!
}

```

## Server Side Implementation
GraphQL has several [server side](https://graphql.org/code/#javascript-server) implementations available in multiple languages. These implementations roughly follow a pipeline pattern with the following stages:

1. an endpoint is exposed which accepts GraphQL queries. 
2. We define a schema with types, query, and mutation. 
3. We associate functions called resolvers with the types in respective programming language to fetch data from underlying systems. 

A graphQL endpoint can live along side Rest APIs. A layered architecture will look like this:

//TODO: Add diagram
As we can the similar to REST , the GraphQL will also depend on a business logic layer for fetching data from underlying systems.

Support for GraphQL constructs varies across implementations. While the basic types: Query and Mutations are supported across all implementations, support for subscription is not available in a few.

## Client Side Implementations

On the client side, we can either send the query as query string in a GET request or as a JSON payload in a POST request;

```shell
curl --request POST 'localhost:0/graphql' --header 'Content-Type: application/json'  --data-raw '{"query":"query {myRecentPurchases(count:10){title,description}}"}'
```
Here we send a request for fetching my 10 recent purchases with the fields title, and description in each record.

In order to avoid making the low-level HTTP calls, we should use a GraphQL client as an abstraction layer to take care of:
1. sending the request and handling the response
2. view layer integrations and optimistic UI updates
3. caching query results

There are several [client frameworks](https://graphql.org/code/#javascript-client) available with popular ones being [Apollo Client](https://github.com/apollographql/apollo-client), [Relay (from Facebook)](https://facebook.github.io/relay/), and [urql](https://github.com/FormidableLabs/urql).

Some flavours of the Query Language:
1. Fetch without arguments
2. Fetch with argument
3. Fragments
4. Variable Type

## GraphQL Example with Spring Boot

We will use a Spring Boot application to build a GraphQL implementation. Let us first create a Spring Boot application with the [Spring Initializr](). 

## Adding the Dependencies
For GraphQL server, we will add the following maven dependencies for a GraphQL starter and java tools module:

```xml
    <dependency>
      <groupId>com.graphql-java</groupId>
      <artifactId>graphql-spring-boot-starter</artifactId>
      <version>5.0.2</version>
    </dependency>
    <dependency>
      <groupId>com.graphql-java</groupId>
      <artifactId>graphql-java-tools</artifactId>
      <version>5.2.4</version>
    </dependency>
```

### Defining the GraphQL Schema 
We can either take a top down approach by defining the schema and then create POJO for each type or a bottom up approach by creating the POJOs first followed by schema from the POJO classes. 

The GraphQL schema needs to be defined in a file with extension `graphqls`. Let us define our schema in a file `product.graphqls`:

```
type Product {
    id: ID!
    title: String!
    description: String!
    category: String
    madeBy: Manufacturer!
}

type Manufacturer {
    id: ID!
    name: String!
    address: String
}

# The Root Query for the application
type Query {
    myRecentPurchases(count: Int, customerID: String): [Product]!
    lastVisitedProducts(count: Int, customerID: String): [Product]!
    productsByCategory(category: String): [Product]!
}

# The Root Mutation for the application
type Mutation {
    addRecentProduct(title: String!, description: String!, category: String) : Product!
}

```
Here we have added three operations to our Query and a Mutation for adding recent products.

Next we define the POJO classes for the Object types: `Product` and `Manufacturer`:

```java
@Data
public class Product {
   private String id; 
   private String title;
   private String description; 
   private String category;
   private Manufacturer madeBy;
}
```
This `Product` POJO maps to the `product` type defined in our GraphQL Schema.

We will now add resolvers for the all the types defined in schema. 

```java
@Service
public class QueryResolver implements GraphQLQueryResolver {

  public List<Product> getMyRecentPurchases(Integer count, String customerID) {
    List<Product> products = new ArrayList<Product>();

    products.add(Product.builder().title("Television").category("Eletronics").description("My new TV").build());
    return products;
  }
  
  public List<Product> getLastVisitedProducts(final Integer count, final String customerID) {
    List<Product> products = new ArrayList<Product>();

    products.add(Product.builder().title("Television").category("Eletronics").description("My new TV").build());
    return products;
  }
  
  public List<Product> getProductsByCategory( final String category) {
    List<Product> products = new ArrayList<Product>();

    products.add(Product.builder().title("Television").category("Eletronics").description("My new TV").build());
    return products;
  }
  
  public List<Product> getRelatedProducts( final String parentProductID) {
    List<Product> products = new ArrayList<Product>();

    products.add(Product.builder().title("Television").category("Eletronics").description("My new TV").build());
    return products;
  }

}
```




### Associate GraphQL Types with Resolvers

Multiple resolver components convert the GraphQl request received from the API consumers or front end and invoke operations to fetch data from applicable data sources. For each type we define a `resolver`.

The resolver of our `Product` type looks like this:

```java
@Service
public class ProductResolver implements GraphQLResolver<Product>{

  public Manufacturer getMadeBy(final Product product) {
    return Manufacturer.builder().id(UUID.randomUUID().toString()).address("gfghfhj").build();
  }
}
```

### Connecting to Datasources
Next we will enable our resolvers to fetch data from underlying datasources like a database or web service. For the purpose of this example, we have configured an in-memory H2 database as the data store for `products` and `manufacturers`. For this, we will use spring JDBC to retrieve data from database and put this logic in a seperate repository classes. 

Apart from fetching data, we can also build different categories of middleware logic in this business service layer, like authorization of incoming request, applying filters on data fetched from backend, transformation into backend data models, and also caching any less frequently changing data.

## Conclusion

In this article, we looked at the main capabilities of GraphQL and how it helps to solve some common problems associated with consuming APIs. 

We also looked at GraphQL's Schema Definition Language (SDL) along with the root types: Query, Mutation and Subscription followed by how it is implemented in the server side with the help of resolver functions. 

We finally set up a GraphQL server implementation with the help of two Spring modules and defined a schema with a Query and Mutation. We then defined resolver functions to connect the query with underlying data source in the form of a H2 database. 

GraphQL is a powerful mechanism for building API based applications: mobile apps and single page applications. However, we should use it to complement REST APIs instead of using it as a complete replacement.
