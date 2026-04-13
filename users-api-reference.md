# Users API reference

Use the WorkHop Users API to manage user accounts in your organisation. 

The API supports the following operations: 
- Inviting new users
- Updating existing user details
- Deleting users

The Users API uses REST conventions and communicates over HTTPS. All request and response bodies use JSON.


### Base URL

```text
https://api.workhop.io/v1
```

### Authorization

OAuth 2.0 is required for all requests to the Users API. Include a valid bearer token in the `Authorization` header of each request.

```text
Authorization: Bearer {your_access_token}
```


### HTTP status codes

The WorkHop Users API uses the following standard HTTP response codes:

| Status code | Description |
|-------------|-------------|
| `200 OK` | The request succeeded. The response body contains the requested or updated resource. |
| `201 Created` | The resource was created successfully. The response body contains the newly created resource. |
| `204 No Content` | The request succeeded. No response body is returned. Returned by DELETE operations. |
| `400 Bad Request` | The request was malformed or missing required fields. Check the request body for missing or incorrectly typed fields and resubmit. |
| `401 Unauthorized` | The request lacks valid authentication credentials. Verify that your OAuth 2.0 bearer token is present and has not expired. |
| `403 Forbidden` | The authenticated user does not have the required permission to perform this action. Verify that the token's scope includes the required permission for this endpoint. |
| `404 Not Found` | The specified resource does not exist. Verify that the `userId` in the request is correct. |
| `409 Conflict` | The request conflicts with an existing resource. For user invitations, this indicates a duplicate email address. Verify that the user does not already exist before retrying. |


### Errors

The WorkHop Users API uses the following error type:

| Error | Description |
|-------|-------------|
| [UserError](#usererror) | Failure in a user-related operation. |


#### UserError

| Field | Type | Description |
|-------|------|-------------|
| `errorType` | enum | Predefined error code. Possible values are `USER_NOT_FOUND`, `DUPLICATE_EMAIL`, `INVALID_ROLE`, and `MISSING_REQUIRED_FIELD`. |
| `errorMessage` | string | Additional information about why the error occurred. |

---

## Users

The Users resource represents individual members of your WorkHop organisation. Use it to manage user accounts, roles, and profile data.


### Data model

| Attribute | Type | Required? | Description |
|-----------|------|-----------|-------------|
| `userId` | string | Required | Unique identifier of the user. |
| `email` | string | Required | Primary email address of the user. Must be unique within the organisation. |
| `firstName` | string | Required | First name of the user. |
| `lastName` | string | Required | Last name of the user. |
| `role` | enum | Required | Role assigned to the user. Possible values are `employee`, `manager`, `hr_admin`, and `executive`. |
| `teamId` | string | Optional | Unique identifier of the team the user belongs to. |
| `attributes` | object | Optional | Key-value pairs of custom attributes assigned to the user. |
| `createdAt` | string | Required | ISO 8601 timestamp of when the user was created. |


<details>
<summary>Example</summary>

```json
{
  "userId": "usr_01h2xk9p3q",
  "email": "eli.morgan@example.com",
  "firstName": "Eli",
  "lastName": "Morgan",
  "role": "employee",
  "teamId": "team_04b7yz2m1r",
  "attributes": {
    "officeLocation": "Montreal",
    "tenured": "Yes"
  },
  "createdAt": "2024-03-15T09:00:00Z"
}
```

</details>


### Endpoints

Use the following endpoints to interact with Users resources.

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | [Invite a user](#invite-a-user) | Invites a new user to the organisation. |
| PATCH | [Update a user](#update-a-user) | Updates an existing user's details. |
| DELETE | [Delete a user](#delete-a-user) | Deletes a user from the organisation. |

---

## Invite a user

Invites a new user to the WorkHop organisation by sending them an email invitation.


### Endpoint

```text
POST /users/invite
```


### Description

Creates a new user record and sends an invitation email to the specified address. The user's account is created with a `pending` status until they accept the invitation.


### Authorization

[OAuth 2.0](#authorization) is required for this request. Calling this endpoint also requires the `users:write` permission.


### Request schema

#### Header parameters

| Header parameter | Type | Required? | Description |
|------------------|------|-----------|-------------|
| `Content-Type` | string | Required | Must be `application/json`. |
| `Authorization` | string | Required | Bearer token for OAuth 2.0 authentication. |


#### Request body

| Field | Type | Required? | Description |
|-------|------|-----------|-------------|
| `email` | string | Required | Email address of the user to invite. Must be unique within the organisation. |
| `firstName` | string | Required | First name of the user. |
| `lastName` | string | Required | Last name of the user. |
| `role` | enum | Required | Role to assign to the user. Possible values are `employee`, `manager`, `hr_admin`, and `executive`. |
| `teamId` | string | Optional | Unique identifier of the team to assign the user to. |


<details>
<summary>Request example</summary>

```json
POST /users/invite
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
Content-Type: application/json

{
  "email": "eli.morgan@example.com",
  "firstName": "Eli",
  "lastName": "Morgan",
  "role": "employee",
  "teamId": "team_04b7yz2m1r"
}
```

</details>


### Response schema

| Status code | Schema | Description |
|-------------|--------|-------------|
| `201 Created` | [User](#data-model) | The user was invited successfully. Returns the created user object. |
| `400 Bad Request` | [UserError](#usererror) | The request was malformed or missing required fields. |
| `409 Conflict` | [UserError](#usererror) | A user with the specified email address already exists. |


<details>
<summary>Response example</summary>

```json
{
  "userId": "usr_01h2xk9p3q",
  "email": "eli.morgan@example.com",
  "firstName": "Eli",
  "lastName": "Morgan",
  "role": "employee",
  "teamId": "team_04b7yz2m1r",
  "attributes": {},
  "createdAt": "2024-03-15T09:00:00Z"
}
```

</details>

---

## Update a user

Updates the details of an existing user.


### Endpoint

```text
PATCH /users/{userId}
```


### Description

Updates one or more fields on an existing user record. Only the fields included in the request body are updated. Fields not included remain unchanged.


### Authorization

[OAuth 2.0](#authorization) is required for this request. Calling this endpoint also requires the `users:write` permission.


### Request schema

#### Path parameters

| Path parameter | Type | Required? | Description |
|----------------|------|-----------|-------------|
| `userId` | string | Required | Unique identifier of the user to update. |


#### Header parameters

| Header parameter | Type | Required? | Description |
|------------------|------|-----------|-------------|
| `Content-Type` | string | Required | Must be `application/json`. |
| `Authorization` | string | Required | Bearer token for OAuth 2.0 authentication. |


#### Request body

| Field | Type | Required? | Description |
|-------|------|-----------|-------------|
| `firstName` | string | Optional | Updated first name of the user. |
| `lastName` | string | Optional | Updated last name of the user. |
| `role` | enum | Optional | Updated role. Possible values are `employee`, `manager`, `hr_admin`, and `executive`. |
| `teamId` | string | Optional | Updated team identifier. |
| `attributes` | object | Optional | Updated key-value pairs of custom attributes. |


<details>
<summary>Request example</summary>

```json
PATCH /users/usr_01h2xk9p3q
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
Content-Type: application/json

{
  "role": "manager",
  "teamId": "team_09c3ab5n2s"
}
```
</details>


### Response schema

| Status code | Schema | Description |
|-------------|--------|-------------|
| `200 OK` | [User](#data-model) | The user was updated successfully. Returns the updated user object. |
| `400 Bad Request` | [UserError](#usererror) | The request was malformed or contained invalid field values. |
| `404 Not Found` | [UserError](#usererror) | No user exists with the specified `userId`. |


<details>
<summary>Response example</summary>

```json
{
  "userId": "usr_01h2xk9p3q",
  "email": "eli.morgan@example.com",
  "firstName": "Eli",
  "lastName": "Morgan",
  "role": "manager",
  "teamId": "team_09c3ab5n2s",
  "attributes": {
    "officeLocation": "Montreal",
    "tenured": "Yes"
  },
  "createdAt": "2024-03-15T09:00:00Z"
}
```
</details>

---

## Delete a user

Deletes a user from the WorkHop organisation.


### Endpoint

```text
DELETE /users/{userId}
```


### Description

Permanently deletes a user record from the organisation. This action cannot be undone.


### Authorization

[OAuth 2.0](#authorization) is required for this request. Calling this endpoint also requires the `users:delete` permission.


### Request schema

#### Path parameters

| Path parameter | Type | Required? | Description |
|----------------|------|-----------|-------------|
| `userId` | string | Required | Unique identifier of the user to delete. |


#### Header parameters

| Header parameter | Type | Required? | Description |
|------------------|------|-----------|-------------|
| `Authorization` | string | Required | Bearer token for OAuth 2.0 authentication. |


<details>
<summary>Request example</summary>

```text
DELETE /users/usr_01h2xk9p3q
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
```
</details>

### Response schema

| Status code | Schema | Description |
|-------------|--------|-------------|
| `204 No Content` | N/A | The user was deleted successfully. |
| `404 Not Found` | [UserError](#usererror) | No user exists with the specified `userId`. |


<details>
<summary>Response example</summary>

```text
204 No Content
```
</details>

---
<small>
  <p>WorkHop is a fictitious B2B SaaS product referenced exclusively in the scope of my portfolio work. Similarities to any real-world products or features is purely coincidental.</p>
</small>

