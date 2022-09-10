# What is REST API and How do they work?

Let's break it into two components:

1. RESTful
2. API

An API is an interface through which one program or web site talks to another.\
They are used to share data and services, and they come in many different
formats and types (eg: JSON or XML data).

REST (Representational State Transfer) describes the general rules (guideline) for how the data and services are represented 
through the API, so that other programs will be able to correctly request and receive the data and services that an API makes available.

An API that complies with some or all of the six guiding rules of REST is considered to be RESTful.

We are able to communicate with servers using the HTTP protocol. 
With these protocols, we can perform CRUD operations (Create, Read, Update and Delete).
When we send the HTTP request, REST picks it up and simplifies the communication process by providing various HTTP methods/operations/verbs which we can use to send requests to the server.

## How to communicate with a server using REST APIs

The most commonly used methods for REST APIs communication process with the servers are:
* GET: To Read data on the server.
* POST: To Create data.
* PATCH/PUT: To Update data.
* DELETE: To Delete data.

These methods allow us to perform CRUD operations
* Create = POST.
* Read = GET.
* Update = PATCH/PUT.
* Delete = DELETE.

If we are to retreive data from a server, we are going to make a GET request to an endpoint provided by the server.
This endpoint is similar to a URL.

If the request is valid, the server will respond with the data we requested and also send a status code 200(success), 400(error).

## Examples of Requests and Responses

We have an application that allows CRUD operation on customers and orders for a business hosted at fashionboutique.com.
We could create an HTTP API that allows a client to perform the operations

To READ/VIEW all customers:
```
GET http://fashionboutique.com/customers
Accept: application/json
```
Response header:
```
Status Code: 200 (OK)
Content-type: application/json

# followed by the requested customers data in application/json format
```

To CREATE a new customer by posting the data:
```
POST http://fashionboutique.com/customers
Body:
{
  “customer”: {
    “name” = “Jone Doe”,
    “email” = “john.doe@someemail.org”
  }
}
```
The server generates an `id` for the object and returns, with a header like:
```
201 (CREATED)
Content-type: application/json

# followed by the data for the customer resource with id 123 in application/json format.
```

To UPDATE/EDIT a customer by `id`:
```
PUT http://fashionboutique.com/customers/123
Body:
{
  “customer”: {
    “name” = “Jone Doe”,
    “email” = “john.doe@someemail.org”
  }
}
```
The server generates a success code like:
```
Status Code: 200 (OK)
```

To DELETE a customer by `id`:
```
DELETE http://fashionboutique.com/customers/123
```
The server generates a response code like:
```
Status Code: 204 (NO CONTENT)
```

