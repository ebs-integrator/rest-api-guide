# REST API Convention in EBS

The purpose of this document is to standardize the process of creating REST API in the EBS Integrator company. To have the same style and level of quality in the applications developed by the teams.

This version of the document is consultative and can be modified for improvement and to come to a common opinion between the frontend and backend teams.

## Documentation

Used documentation tool: **Swagger**

Swagger is an open-source software framework backed by a large ecosystem of tools that helps developers design, build, document, and consume RESTful web services.

Swagger must support setting of Authentication header or other custom headers required by the application

Each endpoint must have a documented request body and response object

## Authentification

Used authentication mechanism: **JWT**

JSON Web Token is an Internet standard for creating data with optional signature and/or optional encryption whose payload holds JSON that asserts some number of claims. The tokens are signed either using a private secret or a public/private key.

**Correct** used header example:

    Authentication: Bearer <token>

**Wrong** header example:

    Authentication: Token <token>
    
    Authentication: <token>

## Date and Time Format

Recommended date format: **ISO 8601**

The ISO 8601 standard is an International Standard for the representation of dates and times. This format contains date, time, and the offset from UTC, as well as the T character that designates the start of the time, for example, 2007-04-05T12:30:22-02:00. The pattern for this date and time format is YYYY-MM-DDThh:mm:ss.sTZD. 

Date: 2007-04-05

Date and time: 2007-04-05T12:30:22Z

Date and time with UTC offset: 2007-04-05T12:30:22+02:00


_Python strptime format: %Y-%m-%dT%H:%M:%S%z_

## Performance

A good response time for a REST API endpoint: 

- **under 100ms is acceptable**, 0.1 second is about the limit for having the user feel that the system is reacting instantaneously
- 1.0 second is about the limit for the user's flow of thought to stay uninterrupted, even though the user will notice the delay
- 10 seconds is about the limit for keeping the user's attention focused on the dialogue

_Based on research: https://www.nngroup.com/books/usability-engineering/_

## Resource naming


### Resource naming

RESTful URI should refer to a resource that is a thing (noun) instead of referring to an action (verb) because nouns have properties which verbs do not have – similar to resources have attributes.


### URL style

URL path is written in case style: kebab-case without a trailing slash.

Kebab case combines lowercase words by replacing each space with a dash (-):

**Correct**:

    GET /call-logs
    GET /call-logs/numbers
    GET /education-programs

**Wrong**:

    GET /Call_logs/
    GET /CallLogs
    GET /Call-logs/


### Resource count

**Plural** if main resources will return a list, singular if the resource will return an object

**Correct**:

    GET /users - endpoint will return a list of users
    GET /users/{id} - endpoint will return an object from list of users
    GET /users/{id}/subscriptions - endpoint will return a list of subscriptions of a specific user
    GET /users/{id}/plan - endpoint will return a plan object of a specific user

**Wrong**:

    GET /user
    GET /user/{id}


### Resource actions

**Correct**:

    POST /users/{id}/block - block a user
    POST /messages/{id}/like - like message
    DELETE /messages/{id}/like - unlike message
    POST /messages/{id}/unlike - alternative unlike message
    POST /friends/invite - invite a friend

**Wrong**:

    GET /users/{id}/block
    POST /block-user/{id}/


## Data format

### Properties name style

Used case style for properties: snake case

Snake case refers to the style of writing in which each space is replaced by an underscore (_) character and the first letter of each word written in lowercase. It is the most common naming convention used in computing as identifiers for variable, function, and file names.

**Correct**:

    {
        "name": "Test",
        "parent_id": 1
    }

**Wrong**:

    {
        "Name": "Test",
        "parentId": 1
    }


### Query parameters

Query params are used only in GET requests for filtering and pagination

**Correct**:

    GET /students/?group=C1020

**Wrong**:

    DELETE /students/?id=1 - removing object by query param
    POST /students/?name=Test&group=C1000 - adding by query params

## Validation

On failed validation, the API will return 400 (Bad Request) status code indicates that the server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax, invalid request message framing, or deceptive request routing).

### Fields validation


If a validation error occurs, the server must return an error for each field that is incorrect

    HTTP 400 Bad Request
    {
        "first_name": [
            "Is required", "Must contain only letters"
        ],
        "age": ["Is required"]
    }
    
If the frontend application has multiple languages, the api must return the error code
    
    HTTP 400 Bad Request
    {
        "first_name": [
            "required", "min_letters"
        ],
        "age": ["required"]
    }

### Generic / non-field validation

If the validation error is not related to a sent field, the api can return the detail property with the message

    HTTP 400 Bad Request
    {
        "detail": "Payment Gateway doesn&#39;t respond"
    }

Multilanguage

    HTTP 400 Bad Request
    {
        "detail": "payment_gateway_no_response"
    }

## Use cases

### Get a list of objects

    GET /students
    
    HTTP 200 OK
    {
        "results": [
            {
                "id": 1,
                "name": "Test",
                "group_id": 1
            }
        ]
    }

### Get a list of objects without results

    GET /students
    
    HTTP 200 OK
    {
        "results": []
    }

### Get a paginated list with results

    GET /students?page=1&page_size=10
    
    HTTP 200 OK
    {
        "count": 1,
        "results": [
            {
                "id": 1,
                "name": "Test",
                "group_id": 1
            }
        ]
    }

### Get a paginated list without results

    GET /students?page=1&page_size=10
    
    HTTP 200 OK
    {
        "count": 0,
        "results": []
    }

### Get a paginated list with additional data

    GET /students?page=1&page_size=10
    
    HTTP 200 OK
    {
        "count": 1,
        "count_active": 1,
        "count_inactive": 0,
        "results": [
            {
                "id": 1,
                "name": "Test",
                "group_id": 1
            }
        ]
    }

### Get a list with a generic search

    GET /students?search=Tes
    
    HTTP 200 OK
    {
        "results": [
            {
                "id": 1,
                "name": "Test",
                "group_id": 1
            }
        ]
    }

### Get an object by id

    GET /students/1
    
    HTTP 200 OK
    {
        "id": 1,
        "name": "Test",
        "group_id": 1
    }

### Get an object by id with nested data

    GET /students/1
    
    HTTP 200 OK
    {
        "id": 1,
        "name": "Test",
        "group": {
            "id": 2,
            "name": "C1000"
        }
    }

### Get an inexistent object by id

    GET /students/1
    
    HTTP 404 OK
    {
        "detail": "Student not found"
    }

### Delete an object

    DELETE /students/1
    
    HTTP 200 OK
    {
        "detail": "Object deleted"
    }

### Delete an object with an error

    DELETE /students/1
    
    HTTP 400 Bad request
    {
    "detail": "Object can't be deleted because of conflict"
    }

### Edit an object

    PUT /students/1
    {
        "name": "Test",
        "group_id": 1
    }
    
    HTTP 200 OK
    {
        "id": 1,
        "name": "Test",
        "group_id": 1
    }

### Patch an object

    PATCH /students/1
    {
        "name": "Test"
    }
    
    HTTP 200 OK
    {
        "id": 1,
        "name": "Test",
        "group_id": 1
    }

### Make an action to an object

    POST /students/1/follow
    
    HTTP 200 OK
    {
        "detail": "Student followed"
    }

### Get referenced resources from an object

    GET /students/1/taxes
    
    HTTP 200 OK
    {
        "results": [
            {
                "id": 1,
                "type": "Test tax",
                "amount": 100
            }
        ]
    }

### Get a specific referenced resource from an object

    GET /students/1/taxes/1
    
    HTTP 200 OK
    {
        "id": 1,
        "type": "Test tax",
        "amount": 100
    }

_Note: In the documentation, this URL with 2 params will have a specific name for each param: ex. GET /students/{student_id}/taxes/{tax_id}_
