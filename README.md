# Webhook API
A simple webhook implementation

## Prerequisites

	- Apache Maven 3+
	- [Martini Desktop](https://www.torocloud.com/martini/download)

## Building the Martini package

```bash
$ mvn clean package
```

This will create a ZIP file named  `webhook-api.zip` containing all the files (services, configurations, etc.) needed under the `target` folder. This ZIP file is what we call a [Martini Package](https://docs.torocloud.com/martini/latest/developing/package/) which then you can import in Martini Desktop to get started. You can learn more how to import a Martini Package by visiting our [documentation](https://docs.torocloud.com/martini/latest/developing/package/importing/)

## Usage

This package exposes operations for a simple Webhook REST API that allows you to do CRUD operations and call the webhook url. You can find the [Gloop REST API](https://docs.torocloud.com/martini/latest/developing/gloop/api/rest/) file named `/webhook/api/Webhook.api` under the `code` folder after importing the package to your Martini Desktop application.

## Setup

This Martini Package automatically sets up the required resources for you. During the package startup, Martini will execute the configured _Startup Service_ that initializes the database.

The package uses HSQL as the default datastore but you can change this to MySQL, Postgres, MSSQL, or Oracle by updating the [package properties](https://docs.torocloud.com/martini/latest/developing/package/properties/) located at [conf](https://docs.torocloud.com/martini/latest/developing/package/directory/) folder. Below is what the contents of the properties file look like for HSQL and MySQL

```
# The database type to be used
database.type=hsql

# HSQL database configuration to be used for testing environment
hsql.driver=org.hsqldb.jdbc.JDBCDriver
hsql.connection.url=jdbc:hsqldb:file:${toroesb.home}data/hsql/<database-to-use>.db;hsqldb.tx=MVCC;sql.syntax_mys=true
hsql.username=sa
hsql.password=

# MySQL database configuration to be used in production environment
mysql.driver=com.mysql.cj.jdbc.Driver
mysql.connection.url=jdbc:mysql://<host>/<database-to-use>
mysql.username=root
mysql.password=
```

* `database.type` sets the database provider the Martini package will use. If you will use the default hsql config, you don't need to change anything in the hsql configuration. **Note**: If you will use a different hsql database, make sure you add `sql.syntax_mys=true` in the connection properties. This ensures that the SQL query from the SQL Services in this package will be compatible with hsql.

## Operations

The base url is `<host>/api/webhooks` where `host` is the location where Martini is deployed. By default, it's `localhost:8080`.

### App

The application that will be used with webhooks.

`POST /app`

Registers a new application.

**Request Body**

`appName` - Name of the application

**Sample Request**

**curl**

```bash
curl -X POST \
	http://localhost:8080/api/webhooks/app \
	-H 'Accept: application/json' \
	-H 'Content-Type: application/json' \
	-d '{
		    "appName": "Application Name"
		}'
```

**Sample Response**

If the request is successful, it will return an HTTP status code of `200` with the response payload below.

```json
{
    "appId": "daed158b-15be-4661-870d-a1c56a4a9545",
    "appName": "Application Name"
}
```

### Event

`GET /event/{eventId}`

Returns an event that matches the given `eventId`

**Sample Request**

**curl**

```bash
curl -X GET \
	http://localhost:8080/api/webhooks/event/bb652732-a072-4d67-8785-aa661ffe08be \
	-H 'Accept: application/json'
```

**Sample Response**

If the request is successful, it will return an HTTP status code of `200` with the response payload below.

```json
{
    "webhook_event_id": "bb652732-a072-4d67-8785-aa661ffe08be",
    "event": "Test event",
    "app": "daed158b-15be-4661-870d-a1c56a4a9545"
}
```

`POST /event`

Creates a new event

**Request Body**

`event` - Name of the webhook event
`app` - appId of the webhook application

**Sample Request**

**curl**

```bash
curl -X POST \
	http://localhost:8080/api/webhooks/event \
	-H 'Accept: application/json' \
	-H 'Content-Type: application/json' \
	-d '{
		    "event": "Test event",
		    "app": "daed158b-15be-4661-870d-a1c56a4a9545"
		}'
```

**Sample Response**

If the request is successful, it will return an HTTP status code of `200` with the response payload below.

```json
{
    "status": "Successfully added new event Test event"
}
```

`DELETE /event/{eventId}`

Deletes an event that matches the given `eventId`

**Sample Request**

**curl**

```bash
curl -X DELETE \
	http://localhost:8080/api/webhooks/event/bb652732-a072-4d67-8785-aa661ffe08be \
	-H 'Accept: application/json'
```

**Sample Response**

If the request is successful, it will return an HTTP status code of `200` with the response payload below.

```json
{
    "status": "Successfully deleted application"
}
```

`GET /events/{appId}`

Returns a list of events by `appId`

**Sample Request**

**curl**

```bash
curl -X GET \
	http://localhost:8080/api/webhooks/events/a7a996dc-b9d9-437b-b15f-60a6f78950d4 \
	-H 'Accept: application/json'
```

**Sample Response**

If the request is successful, it will return an HTTP status code of `200` with the response payload below.

```json
[
    {
        "webhook_event_id": "e6d6e544-d67a-44ce-8d12-02a9540af6cd",
        "event": "Called website",
        "app": "a7a996dc-b9d9-437b-b15f-60a6f78950d4"
    },
    {
        "webhook_event_id": "f99eb66f-59b2-4ad3-b532-d6f25612bcac",
        "event": "User added",
        "app": "a7a996dc-b9d9-437b-b15f-60a6f78950d4"
    },
    ...
]
```

`POST /event/trigger`

Triggers an event and calls all webhooks that references the event.

**Request Body**

`appId` - The id of the webhook application
`payload` - The payload to send by the webhook
`event` - Name of the event to send

**Sample Request**

**curl**

```bash
curl -X POST \
	http://localhost:8080/api/webhooks/event/trigger \
	-H 'Accept: application/json' \
	-H 'Content-Type: application/json' \
	-d '{
	    "appId": "576aa161-428f-4963-847c-b135ff44e432",
	    "payload": {
			"sample": "sample"
	    },
	    "event": "Test event"
	}'
```

**Sample Response**

The request should always be successful as the application should not care if the webhook call was a success or not. To check if the call was as success, check the [tracker](https://docs.torocloud.com/martini/latest/tracker/) for error logs.

```json
{
    "status": "Called webhook"
}
```

### Webhook

`GET /webhook/{appId}`

Returns a list of webhooks by `appId`

**Sample Request**

**curl**

```bash
curl -X GET \
	http://localhost:8080/api/webhooks/webhook/a7a996dc-b9d9-437b-b15f-60a6f78950d4 \
	-H 'Accept: application/json'
```

**Sample Response**

If the request is successful, it will return an HTTP status code of `200` with the response payload below.

```json
[
    {
        "webhook_id": "fd6b3557-8088-4373-ab00-9988d9a988dc",
        "name": "Test Webhook",
        "webhook_url": "http://localhost:8080/api/webhook/test",
        "webhook_event_id": "f99eb66f-59b2-4ad3-b532-d6f25612bcac"
    },
    ...
]
```

`POST /webhook`

Creates a new webhook

**Request Body**

`name` - Name of the webhook
`webhook_url` - Url the webhook will send the payload to
`webhook_event_id` - The id of the webhook event 

**Sample Request**

**curl**

```bash
curl -X POST \
	http://localhost:8080/api/webhooks/webhook \
	-H 'Accept: application/json' \
	-H 'Content-Type: application/json' \
	-d '{
	    "name": "Test webhook",
	    "webhook_url": "http://localhost:8080/api/test",
	    "webhook_event_id": "f99eb66f-59b2-4ad3-b532-d6f25612bcac"
	}'
```

**Sample Response**

If the request is successful, it will return an HTTP status code of `200` with the response payload below.

```json
{
    "webhook_id": "495ce8a6-c79a-4e45-95b7-edfcbe6fbdc3",
    "name": "Test webhook",
    "webhook_url": "http://localhost:8080/api/test",
    "webhook_event_id": "f99eb66f-59b2-4ad3-b532-d6f25612bcac"
}
```

`PUT /webhook`

Updates a webhook that matches the `webhook_id`

**Request Body**

`webhook_id` - The id of the webhook

Required atleast 1 of the following parameters:

`name` - Name of the webhook
`webhook_url` - Url the webhook will send the payload to
`webhook_event_id` - The id of the webhook event

**Sample Request**

**curl**

```bash
curl -X PUT \
	http://localhost:8080/api/webhooks/webhook \
	-H 'Accept: application/json' \
	-H 'Content-Type: application/json' \
	-d '{
	    "webhook_id": "495ce8a6-c79a-4e45-95b7-edfcbe6fbdc3",
	    "name": "Test webhook (Updated)"
	}'
```

**Sample Response**

If the request is successful, it will return an HTP status code of `200` with the response payload below.

```json
{
	"status": "Successfully updated webhook."
}
```

`DELETE /webhook/{webhookId}`

Deletes a webhook that matches the `webhookId`

**Sample Request**

**curl**

```bash
curl -X DELETE \
	http://localhost:8080/api/webhooks/webhook/495ce8a6-c79a-4e45-95b7-edfcbe6fbdc3 \
	-H 'Accept: application/json'
```

**Sample Response**

If the request is successful, it will return an HTTP status code of `200` with the response payload below.

```json
{
	"status": "Deleted webhook with id 495ce8a6-c79a-4e45-95b7-edfcbe6fbdc3"
}
```
