# GraphQL
* used instead of ReSTful endpoints
* APIs are organized in terms of types and fields, not endpoints
* GraphQL creates a uniform API across your entire application without being limited by a specific storage engine

## Results and uses
* reduces boilerlate required to expose new functionality to clients
* improves performance by limiting returned data to only requested data
* exposes richer functcionality by leveraging the nobject graph to provide context-aware functionality deep within the ndomain

## Repository
* Data access layer
* responsible for executing queries against databases, reconstructing JS objects into domain entities for use within your domain layer
  * domain layer is a collection of entity objects and related business logic that is designed to represent the enterprise business model
  *

## Services
* layer that contains conditional logic, control flow, or orchestration
* "adapters" related to the ports (db or trigger specific mappings) should be colated with their repsective layers
* service layer effectively allows you to share code between request handles and async operations, queue consumer
* encourages modular code - functions can be broken down into smaller pieces and composed together to perform larger operations

## Caching
* solves the problem where the backend receives a request for information that you've recently fetched and can return it without another round trip to the database
* heavy caching leveraged between data access layer and service layer, which has been enabled in large part due to the heavy adoption of the repository design pattern
* can seamlessly cache the results of almost any repository call because the business logic has no reliance on the underlying data store

## Batching
* solves the problem where you've received more than one request for the same data and you're still receiving additional requests as you execute the first call
  * ex: two clients request the same data. both requestors will be queued up to the receive the results of the original call
  * when the first call resolves, every queued call gets their results all at once
* **REAL WORLD EXAMPLE OF WHEN BATCHING IS USEFUL??** maybe new product page?

## Data Loader
* in-memory cache object which sits at the request level
* as a request moves throughout the middleware pipeline, request context can be extracted and as data is fetched, it's stored within a loader for a specific API
  * subsequent requests can query the _loader_ rather than the API and retrieve in-memory results if they'be been previously queried

## Controllers

