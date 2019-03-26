# Microservices Chapter 7 - Implementing Queries in a Mircoservices Architecture

Tldr; writing queries for multiple services is hard! You got 2 options.
1. API Composition: Ask all the services and combine the results
  - Simple. Use it when you can.
2. Command Query Responsibility Segregation (CQRS): Maintain a db for this particular query
  - This is more powerful than the API composition pattern, but it’s also more complex.
  - It maintains one or more view databases whose sole purpose is to support queries.

## 7.1 Querying using the API composition pattern

![API Composition](https://i.imgur.com/NV9hOyi.png)

There is an API composer that is in charge of making the requests to each service.

The composer can be something on the front end or backend.

This pattern is not great with large datasets that may need in mem joins.

Question: What's the difference between figure 7.5 and figure 7.6? (gateway vs. standalone service). They seem like the same thing?

Pro Tip: Access all APIs concurrently (did some one say Elixir?) rather than sequentially when possible.

The drawbacks:
- Increased overhead - More DB requests, more network traffic...
- Risk of reduced availability - The more services you're connected to, the more likely one can take you down
- Lack of transactional data consistency

Complex queries (like orders no older than X containing the word Y) would require the API composition pattern to grab a lot of extra data from other services and then join it in memory (non performant).

`findAvailableRestaurants` is a good example. It might seem to make sense that the `RestaurantService` owns this because they have the data, but it's really a much more complex thing that has a different main concern.
- Using the API composition pattern to retrieve data scattered across multiple services results in expensive, inefficient in-memory joins.
- The service that owns the data stores the data in a form or in a database that doesn’t efficiently support the required query.
- The need to separate concerns means that the service that owns the data isn’t the service that should implement the query operation.

pg 231

Command side: (CUD) create/update/delete
Query side: get

The get data stays in sync from events published

_add screenshot_
![Non-CQRS v CQRS](https://i.imgur.com/ZI1mODa.png)

Separate DBs used

Question: Does this run this risk of data getting out of sync? Sounds like you would need to be very careful to ensure that it doesn't?

CQRS benefits are as follows:
- Enables the efficient implementation of queries in a microservice architecture
- Enables the efficient implementation of diverse queries
- Makes querying possible in an event sourcing-based application
- Improves separation of concerns

pg. 235

The drawbacks of CQRS
- More complex architecture
- Dealing with the replication lag

pg. 236

## 7.3 Designing CQRS Views

Choosing a DB is all about the view that you want to be able to create.

![Selecting a db table](https://i.imgur.com/FbFPpVC.png)

Table for deciding what type of DB you'd like to use.

API calls should not directly access the DB. DAO (Data Access Objects) should be used with their helpers:
- Handles concurrency
- Makes sure events are idempotent

Adding to a CQRS may be hard an could require that you need to read through old events (a backfill?) - pg. 242

## 7.4 Implementing a CQRS view with AWS DynamoDB

Detecting duplicate events can help ensure data does not get out of sync.

## Summary

- Implementing queries that retrieve data from multiple services is challenging because each service’s data is private.
- There are two ways to implement these kinds of query: the API composition pattern and the Command query responsibility segregation (CQRS) pattern.
- The API composition pattern, which gathers data from multiple services, is the simplest way to implement queries and should be used whenever possible.
- A limitation of the API composition pattern is that some complex queries require inefficient in-memory joins of large datasets.
- The CQRS pattern, which implements queries using view databases, is more powerful but more complex to implement.
- A CQRS view module must handle concurrent updates as well as detect and discard duplicate events.
- CQRS improves separation of concerns by enabling a service to implement a query that returns data owned by a different service.
- Clients must handle the eventual consistency of CQRS views.
