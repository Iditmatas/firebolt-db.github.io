---
layout: default
title: Service accounts (Beta)
description: Learn about creating service account users for Firebolt.
nav_order: 3
parent: Account and user management
---

# Creating a service account (Beta)
{: .no_toc}

A service account user can access Firebolt programmatically **only**.

## SQL commands for managing service account users

To create a service account user, follow these steps:

1. Create a service account user using the [`CREATE SERVICE ACCOUNT USER` command](#create-a-service-account-user). Make a note of the service account ID - you will need that to authenticate later. The ID can always be retrieved by querying the `service_account_users` view in Firebolt’s information schema - for more information see [Service account users in information schema](#service-account-users-in-information_schema).

2. Generate a secret for the service account user with [the generation function described below](#generate-a-secret-for-the-service-account-user). 
**Make a note of the secret** - it can't be retrieved later.  In case the secret is lost (or needs to be rotated), you can always generate a new secret, calling the same generation function.

3. Use the service account ID and the secret to [access Firebolt programmatically](#authenticate-with-a-service-account-via-the-rest-api) via Firebolt’s REST API.

To delete a service account user, use the [`DROP SERVICE ACCOUNT` command](#delete-a-service-account-user).

## Create a service account user
`CREATE SERVICE ACCOUNT <name> ROLE = <role> [DESCRIPTION = <description>]`

Creates a service account user, where:

| Property                          | Data type | Description |
| :------------------------------   | :-------- | :---------- |
| name                              | TEXT      | The name of the user. |
| role                              | TEXT      | A role assigned to the user. |
| description                       | TEXT      | The description of the user. |

**Example**

`CREATE SERVICE ACCOUNT tableau_user ROLE='viewer' DESCRIPTION='Used for Tableau dashboards'; `

## Generate a secret for the service account user
`CALL firebolt.GENERATESERVICEACCOUNTKEY('<name>');`

Generate a secret for the service account user, where:

| Property                          | Data type | Description |
| :------------------------------   | :-------- | :---------- |
| name                              | TEXT      | The name of the user. |


**Example**
`CALL firebolt.GENERATESERVICEACCOUNTKEY('tableau_user');`

The command returns both the service account ID and secret.

{: .note}
Generating a new secret for your service account user replaces any previous secret (which cannot be used once a new one is generated). Make a note of the secret and keep it in a secured location.

## Delete a service account user
`DROP SERVICE ACCOUNT <name>;`

Deletes a service account user by its name. The name can be retrieved by running the 
`SELECT * FROM INFORMATION_SCHEMA.SERVICE_ACCOUNT_USERS` command, where:

| Property                          | Data type | Description |
| :------------------------------   | :-------- | :---------- |
| name                              | TEXT      | The name of the user. |


**Example**
`DROP SERVICE ACCOUNT tableau_user;`

## Service account users in information_schema
`SELECT * FROM information_schema.service_account_users;`

Returns a list of service account users. 

The command returns the following properties for each service account user:

| Property                          | Data type | Description |
| :------------------------------   | :-------- | :---------- |
| name                              | TEXT      | The name of the user. |
| id                                | TEXT      | The ID of the user. |
| role                              | TEXT      | The role that was assigned to the user. The following values are possible: ‘Viewer,’ ‘DB admin,’ and ‘Account admin.’ The roles can be specified in upper or lower case. For accounts that support custom roles (DB RBAC), those can also be specified. |
| description                       | TEXT      | The description of the user. |
| created_on                        | TIMESTAMP | Time (UTC) that the user was created. |
| last_altered                      | TIMESTAMP | Time (UTC) that the user was last edited. |

**Example**

`SELECT * FROM INFORMATION_SCHEMA.SERVICE_ACCOUNT_USERS; `

**Returns**

| name         | id            | role          | description                | created_on  | last_altered |
| :------------| :------------ | :------------ | :------------------------- | :---------- | :---------- |
| tableau_user | 217-3813-278  | Account Admin | Used for Tableau dasboards | 2021-01-01 12:00:00 | 2021-01-10 13:50:00 |


## Authenticate with a service account via the REST API
To authenticate Firebolt using service accounts via Firebolt’s REST API, send the following request to receive an authentication token:

```json
curl --location --request POST 'https://api.app.firebolt.io/auth/v1/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=<id>' \
--data-urlencode 'client_secret=<secret>' \
--data-urlencode 'grant_type=client_credentials'
```

Where:

| Property                          | Data type | Description |
| :------------------------------   | :-------- | :---------- |
| id                                | TEXT      | The user’s ID ([created here](#create-a-service-account-user)). |
| secret                            | TEXT      | The user’s secret ([generated here](#generate-a-secret-for-the-service-account-user)). |


Use the returned access_token to authenticate with Firebolt.


## Known limitations and future release plans

**IP allowed/blocked lists**
At this time, using IP allowed/blocked lists (Beta) with service account users is not supported. This will be supported in the future. 

**Information_schema running queries view**
At this time, the `user_id` column in the `information_schema.running_queries` view does not contain the service account ID (it contains an empty `TEXT` instead). This will be supported in future versions.





