Below is a comprehensive documentation designed for a React (or frontend) developer who will be consuming these API endpoints. It covers the main functionalities, the request/response structures, expected input/output, and error handling.

---

# Overview

This FastAPI server provides authentication (sign-up, login) and bookmark management (create, read, update, delete) functionalities. It also offers basic user profile management. The server connects to a MongoDB database using the [PyMongo](https://pypi.org/project/pymongo/) library. The key collections are:

1. **Users** (stores user information).
2. **Bookmarks** (stores user-specific bookmarks).

The server allows Cross-Origin Resource Sharing (CORS) for all origins and supports standard CRUD operations through RESTful endpoints.

---

# Base URL

When running locally (e.g., with `uvicorn server:app --host 0.0.0.0 --port 8000`), the base URL will typically be:

```
http://localhost:8000
```

All documented endpoints are prefixed with `/api`.

---

# Table of Contents

1. [Authentication](#authentication)
   - [POST /api/auth/signup](#post-apiauthsignup)
   - [POST /api/auth/login](#post-apiauthlogin)

2. [Bookmarks](#bookmarks)
   - [POST /api/bookmarks](#post-apibookmarks-create-a-new-bookmark)
   - [GET /api/bookmarks](#get-apibookmarks-list-all-bookmarks)
   - [DELETE /api/bookmarks](#delete-apibookmarks-delete-a-bookmark)
   - [PUT /api/bookmarks](#put-apibookmarks-update-an-existing-bookmark)

3. [Users](#users)
   - [GET /api/users](#get-apiusers-get-user-profile)
   - [PUT /api/users](#put-apiusers-update-user-profile)

4. [Error Handling & Status Codes](#error-handling--status-codes)
5. [Example Usage (React)](#example-usage-react)

---

## Authentication

### POST /api/auth/signup

**Description:**  
Registers a new user.

**URL:**  
```
POST /api/auth/signup
```

**Request Body (JSON):**
```json
{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "password": "Secret@123"
}
```
- **name** *(string, required)*: User’s name.  
- **email** *(string, required)*: User’s email.  
- **password** *(string, required)*: User’s chosen password.  

**Success Response (JSON):**
```json
{
  "message": "User registered successfully"
}
```

**Possible Error Responses (JSON)**:
- **400 Bad Request** if any field is missing or fails validation:
  ```json
  { "detail": "Missing fields" }
  ```
  or
  ```json
  { "detail": "User already exists" }
  ```
  or other validation details.

---

### POST /api/auth/login

**Description:**  
Logs in a user by validating email and password.

**URL:**  
```
POST /api/auth/login
```

**Request Body (JSON):**
```json
{
  "email": "john.doe@example.com",
  "password": "Secret@123"
}
```
- **email** *(string, required)*: The user’s email.  
- **password** *(string, required)*: The user’s password.

**Success Response (JSON):**
```json
{
  "_id": "someMongoIdString",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "passwordHash": "...",
  "createdAt": "2025-01-24T12:34:56Z",
  "metadata": {},
  "updatedAt": "2025-01-24T12:34:56Z"
}
```
This returns the user object from the database. The password is stored as `passwordHash`, but you still receive it in the JSON for demonstration. You might choose to ignore or mask that on the client side for security.

**Possible Error Responses (JSON)**:
- **400 Bad Request** if:
  ```json
  { "detail": "Missing fields" }
  ```
  or
  ```json
  { "detail": "No user found" }
  ```
  or other validation details.

---

## Bookmarks

### POST /api/bookmarks (Create a New Bookmark)

**Description:**  
Saves a new bookmark for a specific user.

**URL:**  
```
POST /api/bookmarks
```

**Request Body (JSON):**
```json
{
  "title": "My Favorite Website",
  "url": "https://www.example.com",
  "privacy": "public",
  "authorEmail": "john.doe@example.com"
}
```
- **title** *(string, required)*: Title or name for the bookmark.  
- **url** *(string, required)*: Link/URL for the bookmark.  
- **privacy** *(string, optional, default "public")*: Privacy setting (`public` or `private`).  
- **authorEmail** *(string, required)*: The owner’s email for the bookmark.

**Success Response (JSON):**
```json
{
  "message": "Bookmark registered successfully"
}
```

**Possible Error Responses (JSON)**:
- **400 Bad Request** if fields are missing or the bookmark already exists for this user:
  ```json
  { "detail": "Fields empty" }
  ```
  or
  ```json
  { "detail": "Bookmark already exists" }
  ```

---

### GET /api/bookmarks (List All Bookmarks)

**Description:**  
Lists bookmarks based on the provided `authorEmail` (and optionally `title`).

**URL:**  
```
GET /api/bookmarks
```

> **Note**: This endpoint, as written in the code, expects the query parameters in the *request body*. However, a more typical REST approach would be to send `authorEmail`/`title` as **query parameters** or path parameters. In the current code, it uses `data: dict = Body(...)`, so you must send a JSON body even for a GET request.  

**Request Body (JSON):**
```json
{
  "authorEmail": "john.doe@example.com",
  "title": "My Favorite Website"  // optional
}
```
- **authorEmail** *(string, required)*: The user’s email whose bookmarks are to be fetched.  
- **title** *(string, optional)*: If provided, fetch only bookmarks matching this title.

**Success Response (JSON):**
```json
{
  "data": [
    {
      "_id": "someMongoIdString",
      "title": "My Favorite Website",
      "url": "https://www.example.com",
      "privacy": "public",
      "authorEmail": "john.doe@example.com",
      "authorEmailUrlHashedPrimaryKey": "...",
      ...
    },
    {
      "_id": "anotherMongoIdString",
      "title": "Another Bookmark",
      ...
    }
  ],
  "meta": {
    "total": 2
  }
}
```
- **data**: Array of bookmark objects.  
- **meta**: Contains `total` = number of bookmarks retrieved.

**Possible Error Responses (JSON)**:
- **404 Not Found** if no bookmarks exist for that user:
  ```json
  { "detail": "No bookmarks found for the provided email" }
  ```
- **400 Bad Request** if `authorEmail` is missing.

---

### DELETE /api/bookmarks (Delete a Bookmark)

**Description:**  
Deletes a bookmark based on `authorEmail` and `title`.

**URL:**  
```
DELETE /api/bookmarks
```

**Request Body (JSON):**
```json
{
  "authorEmail": "john.doe@example.com",
  "title": "My Favorite Website"
}
```
- **authorEmail** *(string, required)*: The user’s email.  
- **title** *(string, required)*: The title of the bookmark to delete.

**Success Response (JSON):**
```json
{
  "message": "Bookmark deleted successfully",
  "meta": {
    "deleted_count": 1
  }
}
```

**Possible Error Responses (JSON)**:
- **404 Not Found** if no matching bookmark found:
  ```json
  { "detail": "No matching bookmark found to delete" }
  ```
- **400 Bad Request** if `authorEmail` or `title` is missing.

---

### PUT /api/bookmarks (Update an Existing Bookmark)

**Description:**  
Updates a bookmark by `authorEmail` and `title`. It only updates the fields you provide in `to_update`.

**URL:**  
```
PUT /api/bookmarks
```

**Request Body (JSON):**
```json
{
  "authorEmail": "john.doe@example.com",
  "title": "My Favorite Website",
  "to_update": {
    "privacy": "private",
    "title": "My Renamed Bookmark"
  }
}
```
- **authorEmail** *(string, required)*: Owner’s email.  
- **title** *(string, required)*: Current bookmark title to match.  
- **to_update** *(object, required)*: Key-value pairs of fields to update.  

**Success Response (JSON):**
```json
{
  "message": "1 bookmarks updated successfully",
  "updated_fields": {
    "privacy": "private",
    "title": "My Renamed Bookmark"
  }
}
```
- The `message` shows how many bookmarks were updated.

**Possible Error Responses (JSON)**:
- **404 Not Found** if no bookmark with that authorEmail and title is found:
  ```json
  { "detail": "No bookmarks found for the given email and title" }
  ```
- **400 Bad Request** if `to_update` is empty or if it contains fields not in the original bookmark:
  ```json
  { "detail": "Some fields in 'to_update' do not exist in the bookmark" }
  ```
- **500 Internal Server Error** if an unexpected error occurs.

---

## Users

### GET /api/users (Get User Profile)

**Description:**  
Fetches a user’s profile by their email.

**URL:**  
```
GET /api/users
```

> **Note**: Like the bookmarks `GET`, this also expects a JSON body in the code snippet.  

**Request Body (JSON):**
```json
{
  "email": "john.doe@example.com"
}
```
- **email** *(string, required)*: The user’s email to look up.

**Success Response (JSON):**
```json
{
  "_id": "someMongoIdString",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "passwordHash": "...",
  "createdAt": "2025-01-24T12:34:56Z",
  "metadata": {},
  "updatedAt": "2025-01-24T12:34:56Z"
}
```

**Possible Error Responses (JSON)**:
- **404 Not Found** if user doesn’t exist:
  ```json
  { "detail": "User not found" }
  ```

---

### PUT /api/users (Update User Profile)

**Description:**  
Updates user fields specified in `to_update`.

**URL:**  
```
PUT /api/users
```

**Request Body (JSON):**
```json
{
  "email": "john.doe@example.com",
  "to_update": {
    "name": "John Updated",
    "metadata": {
      "role": "admin"
    }
  }
}
```
- **email** *(string, required)*: The user’s email.  
- **to_update** *(object, required)*: Key-value pairs of fields to update in the user document.  

**Success Response (JSON):**
```json
{
  "message": "User updated successfully",
  "meta": {
    "modified_count": 1,
    "updated_fields": {
      "name": "John Updated",
      "metadata": {
        "role": "admin"
      }
    }
  }
}
```

**Possible Error Responses (JSON)**:
- **404 Not Found** if user doesn’t exist:
  ```json
  { "detail": "User not found" }
  ```
- **400 Bad Request** if no fields are provided in `to_update`, or an invalid field is provided:
  ```json
  { "detail": "Some fields in 'to_update' do not exist in the user document" }
  ```
- **500 Internal Server Error** if an unexpected error occurs.

---

## Error Handling & Status Codes

Below are the main HTTP status codes used:

- **200 OK** – The request has succeeded.
- **201 Created** – Resource successfully created (not currently returned, but implied).
- **400 Bad Request** – The request could not be understood (missing or invalid fields).
- **404 Not Found** – The resource could not be found (user/bookmark not in DB).
- **500 Internal Server Error** – A server-side error occurred (usually an exception in Python code).

The response body for error cases typically has the format:
```json
{
  "detail": "Explanation of error"
}
```

---

## Example Usage (React)

Below is a minimal example of how you might interact with these endpoints in React using **fetch** or **axios**. Adjust it to your app’s structure and error handling needs.

### 1. Sign Up User

```js
// Using fetch
const signUp = async (name, email, password) => {
  try {
    const response = await fetch('http://localhost:8000/api/auth/signup', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name, email, password })
    });
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.detail);
    }
    
    const data = await response.json();
    console.log(data.message); // "User registered successfully"
  } catch (error) {
    console.error('Error signing up:', error);
  }
};
```

### 2. Log In User

```js
const logIn = async (email, password) => {
  try {
    const response = await fetch('http://localhost:8000/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.detail);
    }

    const userData = await response.json();
    console.log('Logged in user:', userData);
  } catch (error) {
    console.error('Error logging in:', error);
  }
};
```

### 3. Create Bookmark

```js
const createBookmark = async (bookmarkData) => {
  try {
    const response = await fetch('http://localhost:8000/api/bookmarks', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(bookmarkData)
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.detail);
    }

    const data = await response.json();
    console.log(data.message);
  } catch (error) {
    console.error('Error creating bookmark:', error);
  }
};

// Example usage:
createBookmark({
  title: 'My Favorite Website',
  url: 'https://example.com',
  privacy: 'public',
  authorEmail: 'john.doe@example.com'
});
```

### 4. List Bookmarks

```js
const getBookmarks = async (authorEmail, title="") => {
  try {
    // Note: It's a GET request, but code requires a JSON body in this snippet
    const response = await fetch('http://localhost:8000/api/bookmarks', {
      method: 'GET',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ authorEmail, title })
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.detail);
    }
    
    const data = await response.json();
    console.log('Bookmarks:', data);
  } catch (error) {
    console.error('Error fetching bookmarks:', error);
  }
};

// Example usage:
getBookmarks('john.doe@example.com');
```

### 5. Delete Bookmark

```js
const deleteBookmark = async (authorEmail, title) => {
  try {
    const response = await fetch('http://localhost:8000/api/bookmarks', {
      method: 'DELETE',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ authorEmail, title })
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.detail);
    }

    const data = await response.json();
    console.log(data.message);
  } catch (error) {
    console.error('Error deleting bookmark:', error);
  }
};
```

### 6. Update Bookmark

```js
const updateBookmark = async (authorEmail, title, toUpdate) => {
  try {
    const response = await fetch('http://localhost:8000/api/bookmarks', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ authorEmail, title, to_update: toUpdate })
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.detail);
    }

    const data = await response.json();
    console.log(data.message, data.updated_fields);
  } catch (error) {
    console.error('Error updating bookmark:', error);
  }
};

// Example usage:
updateBookmark('john.doe@example.com', 'My Favorite Website', { privacy: 'private' });
```

### 7. Get User Profile

```js
const getUserProfile = async (email) => {
  try {
    const response = await fetch('http://localhost:8000/api/users', {
      method: 'GET',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email })
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.detail);
    }

    const userData = await response.json();
    console.log('User profile:', userData);
  } catch (error) {
    console.error('Error fetching user profile:', error);
  }
};
```

### 8. Update User Profile

```js
const updateUserProfile = async (email, updates) => {
  try {
    const response = await fetch('http://localhost:8000/api/users', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, to_update: updates })
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.detail);
    }

    const data = await response.json();
    console.log(data.message, data.meta);
  } catch (error) {
    console.error('Error updating user profile:', error);
  }
};

// Example usage:
updateUserProfile('john.doe@example.com', { name: 'John Updated' });
```

---

# Final Notes

- **CORS**: All origins (`*`) are allowed by default. If you need stricter control, update the `allow_origins`, `allow_credentials`, `allow_methods`, and `allow_headers` in the `CORSMiddleware` configuration.
- **Validation**: The backend code includes basic validation for name, email, and password during sign-up. You can extend it as needed.
- **Security**: 
  - User passwords are hashed before being stored in the database.  
  - For production, consider implementing JWT-based authentication instead of returning the entire user record on login.  
  - Typically, you’d not send the `passwordHash` back to the client.  

This documentation should give you everything you need to integrate your React frontend with the provided FastAPI backend. If anything changes in the backend code (e.g., more advanced routing or additional validations), make sure to update this documentation accordingly.
