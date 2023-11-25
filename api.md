# API Documentation

## Terminology

| Term | Description |
|------|-------------|
| SLUG | SLUG refers to the end of a URL after `/`, identifying the specific post. |
| Absolute URL | Includes the absolute path and query components. |
| Complete URL | Includes scheme, host, port, path, and query components. |

## Expected Behavior
- **Supported Methods**
  - Supports `GET`, `POST`, `PATCH`, and `DELETE` methods.
- **Response Handling**
  - `GET` endpoints deliver requested resources.
  - `GET` and `PATCH` endpoints return updated or newly created resources.
  - `DELETE` endpoints confirm successful deletion.
- **Error Handling**
  - All endpoints provide error messages on failure.
- **Individual Field Update**
  - `PATCH` endpoints enable updating individual fields.
- **Access Authorization**
  - Requires proper authorization tokens for access.
- **Authorization Exceptions**
  - Certain endpoints (`GET /login`, `GET /reset_password`, `GET /images/*`) are accessible without authorization.
- **Bot Protection**
  - Certain endpoints (`POST /login`, `POST /reset_password`) are accessible with bot protection measures.
- **One-Time Token Requirement**
  - Access to `PATCH /reset_password` requires a one-time token.
- **New User Requirements**
  - Existing users create new users who must reset their password initially.
- **Resource Deletion**
  - Deleting a resource also removes related references (Foreign Keys).
- **Image Deletion Constraint**
  - Prevents deletion of images with existing references (Foreign Keys).
- **Logging**
  - Logs activities to syslog.
- **Logging Activities**
  - Logging of IP address and User-Agent when attempts to log in and attempts to reset the password.
  - Logging of IP address, User-Agent and User ID when a user logs in, reset the password, creates, updates, or deletes.

## Endpoints

### Users
- **GET `/users`**
  - Provides a list of users.
- **POST `/users`**
  - Creates a new user and initiates a password reset by sending an email.
- **GET `/users/[uid]`**
  - Retrieves a specific user.
- **PATCH `/users/[uid]`**
  - Modifies the details of a user.
- **DELETE `/users/[uid]`**
  - Removes a user from the system.

### Blogs
- **GET `/blogs`**
  - Fetches a collection of blogs.
- **POST `/blogs`**
  - Adds a new blog post.
- **GET `/blogs/[id]`**
  - Retrieves a specific blog post using its unique identifier or slug.
- **PATCH `/blogs/[uid]`**
  - Updates the content of a blog post.
- **DELETE `/blogs/[uid]`**
  - Deletes a specific blog post.

### Events
- **GET `/events`**
  - Fetches a collection of events.
- **POST `/events`**
  - Creates a new event.
- **GET `/events/[id]`**
  - Retrieves details about a specific event using its unique identifier or slug.
- **PATCH `/events/[uid]`**
  - Updates the details of an event.
- **DELETE `/events/[uid]`**
  - Deletes a specific event.

### Images
- **GET `/images`**
  - Fetches a collection of images.
- **POST `/images`**
  - Uploads a new image.
- **GET `/images/[uid]`**
  - Retrieves a specific image.
- **PATCH `/images/[uid]`**
  - Updates the details of an image.
- **DELETE `/images/[uid]`**
  - Removes a specific image.

### Other Functionalities
- **GET `/location/[address]`**
  - Provides location results based on the provided address.
- **GET `/login`**
  - Offers login validation options, such as captcha.
- **POST `/login`**
  - Authenticates the user and returns authorization information.
- **GET `/reset_password`**
  - Provides password reset validation options, e.g., captcha.
- **POST `/reset_password`**
  - Initiates a password reset and responds with a success message.
- **POST `/reset_password/[token]`**
  - Resets the user's password based on the provided token.

## Images

### Image Sets

| Image Set | Purpose | Minimum | Maximum | Image Format | Example |
|-----------|---------|---------|---------|--------------|---------|
| Original Image | The original uploaded image, functioning as a backup and reference, e.g., for Google Image Bot. | 380x200 pixels | - | - | `/images/<SHA2-224>.ext` |
| Web Content Image | Images optimized for use on the website. | 380x200 pixels | 1920x1005 pixels | - | `/images/optimized/<SHA2-224>.ext` |
| Open Graph Image | Images optimized for use on Facebook marked with the "-og" suffix. | 380x200 pixels | 1920x1005 pixels | 1.9:1 (+-0.09) | `/images/optimized/<SHA2-224>-og.ext` |
| Twitter Cards Image | Images optimized for use on Twitter Cards marked with the "-x" suffix. | 200x200 pixels | 1280x1280 pixels | 1:1 | `/images/optimized/<SHA2-224>-x.ext` |
 
### Image Handling

**Naming:**

- Images are named with a SHA2-224 value followed by an extension type.
- The SHA2-224 value and extension type are determined after the image is optimized but before image processing.

**Storage:**
- An image is stored as an image set [Image Set], where all images in a set have the same SHA2-224 value and extension type.
- The original, unprocessed image is stored in `/images`.
- Optimized and processed images are stored in `/images/optimized`.
- Images are stored in the image format that fits their extension type.
- If an image name already exists and there is a difference between the new original, unprocessed image and the existing original, unprocessed image, the new image is saved as an original, unprocessed image without creating a new image set. The new image should have the suffix '-vV' inserted between the SHA2-224 value and the extension type. 'V' represents the next unused integer starting from 2.

**Optimization and Processing:**

- **Optimization:**
  - Optimized images should be without metadata and undergo TinyPNG optimization.
  - Optimized images should have a DPI of 95.

- **Adjustment of Image Format:** 
  - Adjustment occurs by cropping with a focus on the center of the image to achieve the desired format.

- **Enlargement:**
  - Images should not be enlarged to achieve a size.

### Upload
  - Uploaded images must not exceed 8 MB in size.
  - Permitted formats are JPEG and PNG.
  - Images that do not meet the minimum size are rejected.

## Status Code

**2xx (Success):** Indicate that the request was received, understood, and accepted.
  - **200 OK:** The request was successfully processed and returning the content.
  - **201 Created:** Indicates successful creation of a new resource and returning the content.
  - **204 No Content:** The server successfully processed the request but is not returning any content.

**4xx (Client Errors):** Occur when the client's request contains incorrect syntax or cannot be fulfilled.
  - **400 Bad Request:** The request cannot be fulfilled due to bad syntax or other client-side errors.
  - **401 Unauthorized:** The client needs to authenticate itself to get the requested response.
  - **403 Forbidden:** The client is authenticate but does not have permission to access the requested resource.
  - **404 Not Found:** The requested resource does not exist on the server.

**5xx (Server Errors):** Indicate that the server failed to fulfill a valid request due to an error on its end.
  - **500 Internal Server Error:** A generic error message indicating that something has gone wrong on the server.
  - **503 Service Unavailable:** The server is not ready to handle the request due to temporary overloading or maintenance.

## Main Object

| Data Type | Field | No Content | On Error | Description |
|-----------|-------|------------|----------|-------------|
| Boolean | `success` | Yes | Yes | Indicates if there was an error. |
| Object | `data` | No | No | Contains a page or resource object. |
| Object | `error` | - | Yes | Error object. |
| String | `message` | Yes | No | Describes the status. |

**Error Object:**

| Data Type | Field | Description |
|-----------|-------|-------------|
| String | `code` | Status code. |
| String | `message` | Status as text. |
| String | `description` | Error description. |

**Success Message:**

| Method | Message |
|--------|---------|
| `GET` | Successfully retrieved the resource. |
| `POST` | The resource has been created successfully. |
| `PATCH` | The resource has been updated successfully. |
| `DELETE` | The resource has been deleted successfully. |

**Error Message:**

| Method | Message |
|--------|---------|
| `GET`, `PATCH`, `DELETE` | The requested resource was not found. |
| `POST`, `PATCH` | The request cannot be fulfilled due to bad syntax. |

Examples:

`GET`: Successfully retrieved the resource.
```json
{
  "success": true,
  "data": {
    "uid": 0,
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "createdAt": "2023-11-01T12:00:00Z",
    "updatedAt": "2023-11-15T08:30:00Z",
    "loggedInAt": "2023-11-15T08:30:00Z",
    "_links": {
      "self": { "href": "/users/0", "title": "John Doe" }
    }
  },
  "message": "Successfully retrieved the resource."
}
```

`POST`: The resource has been created successfully.
```json
{
  "success": true,
  "data": {
    "uid": 1,
    "firstName": "Jane",
    "lastName": "Smith",
    "email": "jane@example.com",
    "createdAt": "2023-11-01T12:00:00Z",
    "updatedAt": "2023-11-23T10:45:00Z",
    "loggedInAt": "2023-11-23T10:45:00Z",
    "_links": {
      "self": { "href": "/users/1", "title": "Jane Smith" }
    }
  },
  "message": "The resource has been created successfully."
}
```

`PATCH`: The resource has been updated successfully.
```json
{
  "success": true,
  "data": {
    "uid": 0,
    "firstName": "John",
    "lastName": null,
    "email": "john@example.com",
    "createdAt": "2023-11-01T12:00:00Z",
    "updatedAt": "2023-11-26T10:03:00Z",
    "loggedInAt": "2023-11-26T10:03:00Z",
    "_links": {
      "self": { "href": "/users/0", "title": "John" }
    }
  },
  "message": "The resource has been updated successfully."
}
```

`DELETE`: The resource has been deleted successfully.
```json
{
  "success": true,
  "message": "The resource has been deleted successfully."
}
```

`GET`, `PATCH`, `DELETE`: The requested resource was not found.
```json
{
  "success": false,
  "error": {
    "code": 404,
    "message": "The requested resource was not found.",
    "description": "User with 345 was not found."
  }
}
```

`POST`, `PATCH`: The request cannot be fulfilled due to bad syntax.
```json
{
  "success": false,
  "error": {
    "code": 400,
    "message": "The request cannot be fulfilled due to bad syntax.",
    "description": "firstName is required."
  }
}
```

## List Resource Objects

**List Object:**

| Data Type | Field | Description | Example |
|-----------|-------|-------------|---------|
| List | `items` | A list containing user, blog, or event objects [List Item Objects]. | See examples below. |
| Object | `pagination` | Contains a `next` link. <br> `next` has a null value if there is no next page. | `"pagination": { "next": null }` |
| Number | `results` | Number of results | `"results": 10` |
| Number | `totalResults` | Total number of results | `"totalResults": 150` |
| Number | `page` | Current page | `"page": 1` |
| Number | `totalPages` | Total number of pages | `"totalPages": 15` |

Example:

```json
{
  "items": [
    {
      "uid": 0,
      "firstName": "John",
      "lastName": "Doe",
      "email": "john@example.com",
      "createdAt": "2023-11-01T12:00:00Z",
      "updatedAt": "2023-11-15T08:30:00Z",
      "loggedInAt": "2023-11-15T08:30:00Z",
      "_links": {
        "self": { "href": "/users/0", "title": "John Doe" }
      }
    },
    // Other users...
  ],
  "pagination": {
    "next":  { "href": "/users/?page=2", "title": "Next page" }
  },
  "results": 10,
  "totalResults": 150,
  "page": 1,
  "totalPages": 15
}
```

**Query Parameters:**

| Data Type | Parameter | Description | Example |
|-----------|-----------|-------------|---------|
| Number | `page` | Specifies the page number for paginated results. | `page=2` |
| Number | `max` | Specifies the maximum number of items per page. | `max=20` |

### List Item Objects

**User Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Number | `uid` | No | Unique identifier for the user. |
| String | `firstName` | No | User's first name. |
| String | `lastName` | Yes | User's last name. |
| String | `email` | No | User's email address. |
| String | `createdAt` | No | ISO 8601 representation when the user was created. |
| String | `updatedAt` | Yes | ISO 8601 representation when the user was last updated. |
| String | `loggedInAt` | Yes | ISO 8601 representation when the user last logged in. |
| Object | `_links` | No | Contains a link object for self-reference (`self`). |

Example:

```json
{
  "uid": 0,
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "createdAt": "2023-11-01T12:00:00Z",
  "updatedAt": "2023-11-15T08:30:00Z",
  "loggedInAt": "2023-11-15T08:30:00Z",
  "_links": {
    "self": { "href": "/users/0", "title": "John Doe" }
  }
}
```

Query Parameters:

| Data Type | Parameter | Fields | Description | Example |
|-----------|-----------|--------|-------------|---------|
| String | `initial_letter` | `firstname` | Filters the users by the initial letter of their first name.| `initial_letter=a` (Includes only items starting with 'a') |
| String | `order` | `firstname`, `lastname`, `email`, `createdat`, `updatedat`, `loggedinat` | Specifies the field to sort the user results. | `order=firstname` (Sorts by the `firstname` field) |

**Blog Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Number | `uid` | No | Unique identifier for the blog. |
| String | `title` | No | Title of the blog post. |
| String | `shortDesc` | No | A brief description of the blog post. |
| Object | `image` | No | Image object or a null value. |
| String | `createdAt` | No | ISO 8601 representation when the blog was created. |
| String | `updatedAt` | Yes | ISO 8601 representation when the blog was last updated. |
| Object | `_links` | No | Contains link objects for self-reference (`self`) and slug (`slug`). |

Example:

```json
{
  "uid": 0,
  "title": "Blog Post",
  "shortDesc": "A brief description of the blog post.",
  "image": { "src": "blog_image.jpg", "alt": "Blog Image" },
  "createdAt": "2023-11-10T09:45:00Z",
  "updatedAt": "2023-11-10T10:30:00Z",
  "_links": {
    "self": { "href": "/blogs/0", "title": "Blog Post" },
    "slug": { "href": "/blogs/blog-post", "title": "Blog Post" }
  }
}
```

Query Parameters:

| Data Type | Parameter | Fields | Description | Example |
|-----------|-----------|--------|-------------|---------|
| String | `initial_letter` | `title` | Filters the blog by the initial letter of the title. | `initial_letter=a` (Includes only items starting with 'a') |
| String | `order` | `title`, `createdat`, `updatedat` | Specifies the field to sort the blog results. | `order=title` (Sorts by the `title` field) |

**Event Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Number | `uid` | No | Unique identifier for the event. |
| String | `title` | No | Title of the event. |
| String | `slug` | No | Slug identifying the event. |
| String | `shortDesc` | No | A brief description of the event. |
| String | `startDateTime` | No | ISO 8601 representation the start date and time of the event. |
| String | `location` | No | Location where the event takes place. |
| String | `createdAt` | No | ISO 8601 representation when the event was created. |
| String | `updatedAt` | Yes | ISO 8601 representation when the event was last updated. |
| Object | `_links` | No | Contains link objects for self-reference (`self`) and slug (`slug`). |

Example:

```json
{
  "uid": 0,
  "title": "Tech Conference",
  "slug": "tech-conference",
  "shortDesc": "Annual tech conference.",
  "startDateTime": "2023-12-01T09:00:00Z",
  "location": "Convention Center",
  "createdAt": "2023-10-20T14:00:00Z",
  "updatedAt": "2023-11-05T16:30:00Z",
  "_links": {
    "self": { "href": "/events/0", "title": "Tech Conference" },
    "slug": { "href": "/events/tech-conference", "title": "Tech Conference" }
  }
}
```

Query Parameters:

| Data Type | Parameter | Fields | Description | Example |
|-----------|-----------|--------|-------------|---------|
| String | `initial_letter` | `title` | Filters the event by the initial letter of the title. | `initial_letter=a` (Includes only items starting with 'a') |
| String | `order` | `title`, `startdatetime`, `createdat`, `updatedat` | Specifies the field to sort the event results. | `order=title` (Sorts by the `title` field) |

**Image Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Number | `uid` | No | Unique identifier for the image. |
| String | `src` | No | Image name with extension. |
| String | `alt` | No | Alternate text for the image. |
| String | `createdAt` | No | ISO 8601 representation when the image was created. |
| String | `updatedAt` | Yes | ISO 8601 representation when the image was last updated. |
| Object | `_links` | No | Contains a link object for self-reference (`self`). |

Example:

```json
{
  "uid": 0,
  "src": "example1.jpg",
  "alt": "Example 1",
  "createdAt": "2023-11-02T11:00:00Z",
  "updatedAt": "2023-11-02T11:05:00Z",
  "_links": {
    "self": { "href": "/images/0", "title": "Example 1" }
  }
}
```

Query Parameters:

| Data Type | Parameter | Fields | Description | Example |
|-----------|-----------|--------|-------------|---------|
| String | `order` | `src`, `createdat`, `updatedat` | Specifies the field to sort the image results. | `order=src` (Sorts by the `src` field) |

**Location Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Unknown | <location result fields> | | Not currently defined. |

## Resource Objects

**User Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Number | `uid` | No | Unique identifier for the user. |
| String | `firstName` | No | User's first name. |
| String | `lastName` | Yes | User's last name. |
| String | `email` | No | User's email address. |
| String | `createdAt` | No | ISO 8601 representation when the user was created. |
| String | `updatedAt` | Yes | ISO 8601 representation when the user was last updated. |
| String | `loggedInAt` | Yes | ISO 8601 representation when the user last logged in. |
| Object | `_links` | No | Contains link objects for self-reference (`self`), update (`update`), and delete (`delete`). |

Example:

```json
{
  "uid": 0,
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "createdAt": "2023-11-01T12:00:00Z",
  "updatedAt": "2023-11-15T08:30:00Z",
  "loggedInAt": "2023-11-15T08:30:00Z",
  "_links": {
    "self": { "href": "/users/0", "title": "John Doe" },
    "update": { "href": "/users/0", "title": "Update User", "method": "PATCH" },
    "delete": { "href": "/users/0", "title": "Delete User", "method": "DELETE" }
  }
}
```

**Blog Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Number | `uid` | No | Unique identifier for the blog. |
| String | `title` | No | Title of the blog post. |
| String | `slug` | No | Slug identifying the blog. |
| String | `shortDesc` | No | A brief description of the blog post. |
| Object | `image` | No | Image Referring Object or a null value. |
| String | `content` | No | Content of the blog post in HTML format. |
| String | `createdAt` | No | ISO 8601 representation when the blog was created. |
| String | `updatedAt` | Yes | ISO 8601 representation when the blog was last updated. |
| Object | `_links` | No | Contains link objects for self-reference (`self`), slug (`slug`), update (`update`), and delete (`delete`). |

Example:

```json
{
  "uid": 0,
  "title": "Blog Post",
  "slug": "blog-post",
  "shortDesc": "A brief description of the blog post.",
  "image": { "src": "blog_image.jpg", "alt": "Blog Image" },
  "content": "<p>This is the blog content in HTML format.</p>",
  "createdAt": "2023-11-10T09:45:00Z",
  "updatedAt": "2023-11-10T10:30:00Z",
  "_links": {
    "self": { "href": "/blogs/0", "title": "Blog Post" },
    "slug": { "href": "/blogs/blog-post", "title": "Blog Post" },
    "update": { "href": "/blogs/0", "title": "Update Blog", "method": "PATCH" },
    "delete": { "href": "/blogs/0", "title": "Delete Blog", "method": "DELETE" }
  }
}
```

**Event Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Number | `uid` | No | Unique identifier for the event. |
| String | `title` | No | Title of the event. |
| String | `slug` | No | Slug identifying the event. |
| String | `shortDesc` | No | A brief description of the event. |
| String | `startDateTime` | No | ISO 8601 representation the start date and time of the event. |
| String | `endDateTime` | No | ISO 8601 representation the end date and time of the event. |
| String | `location` | No | Location where the event takes place. |
| String | `content` | No | Details of the event in HTML format. |
| String | `createdAt` | No | ISO 8601 representation when the event was created. |
| String | `updatedAt` | Yes | ISO 8601 representation when the event was last updated. |
| Object | `_links` | No | Contains link objects for self-reference (`self`), slug (`slug`), update (`update`), and delete (`delete`). |

Example:

```json
{
  "uid": 0,
  "title": "Tech Conference",
  "slug": "tech-conference",
  "shortDesc": "Annual tech conference.",
  "startDateTime": "2023-12-01T09:00:00Z",
  "endDateTime": "2023-12-03T18:00:00Z",
  "location": "Convention Center",
  "content": "<p>Event details in HTML format.</p>",
  "createdAt": "2023-10-20T14:00:00Z",
  "updatedAt": "2023-11-05T16:30:00Z",
  "_links": {
    "self": { "href": "/events/0", "title": "Tech Conference" },
    "slug": { "href": "/events/tech-conference", "title": "Tech Conference" },
    "update": { "href": "/events/0", "title": "Update Event", "method": "PATCH" },
    "delete": { "href": "/events/0", "title": "Delete Event", "method": "DELETE" }
  }
}
```

**Image Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Number | `uid` | No | Unique identifier for the image. |
| String | `src` | No | Image name with extension. |
| String | `alt` | No | Alternate text for the image. |
| String | `createdAt` | No | ISO 8601 representation when the image was created. |
| String | `updatedAt` | Yes | ISO 8601 representation when the image was last updated. |
| Object | `_links` | No | Contains link objects for self-reference (`self`), update (`update`), and delete (`delete`). |

Example:

```json
{
  "uid": 0,
  "src": "example1.jpg",
  "alt": "Example 1",
  "createdAt": "2023-11-02T11:00:00Z",
  "updatedAt": "2023-11-02T11:05:00Z",
  "_links": {
    "self": { "href": "/images/0", "title": "Example 1" },
    "update": { "href": "/images/0", "title": "Update Image", "method": "PATCH" },
    "delete": { "href": "/images/0", "title": "Delete Image", "method": "DELETE" }
  }
}
```

**Login Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Unknown | \<bot validation fields> | | Not currently defined. |
| Object | `_links` | No | Contains link `login` |

**Authorization Information Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Unknown | \<authorization information fields> | | Not currently defined. |

**Reset Password Object:**

| Data Type | Field | Null | Description |
|-----------|-------|------|-------------|
| Unknown | \<bot validation fields> | | Not currently defined. |
| Object | `_links` | No | Contains link `reset` |

## POST Body Objects

**User Object:**

| Data Type | Field | Min | Max | Required | Description |
|-----------|-------|-----|-----|----------|-------------|
| String | `firstName` | 2 | 50 | Yes | First name of the user. |
| String | `lastName` | 2 | 50 | No | Last name of the user. |
| String | `email` | 5 | 100 | Yes | <u>Unique email address.</u> |

Example:

```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "johndoe@example.com"
}
```

**Blog Object:**

| Data Type | Field | Min | Max | Required | Description |
|-----------|-------|-----|-----|----------|-------------|
| String | `title` | 5 | 100 | Yes | Title of the blog post. |
| String | `slug` | 5 | 150 | Yes | <u>Unique slug in blogs.</u> |
| String | `shortDesc` | 145 | 155 | Yes | Short description of the blog post. |
| Object | `image` | - | 8MB | No | Image data specified as a data URI scheme, encoded in base64. |
| String | `content` | 300 | 65,535 | Yes | Content of the blog post in HTML format. |

Example:

```json
{
  "title": "Example Blog Title",
  "slug": "example-blog-title",
  "shortDesc": "This is a short description of the blog post.",
  "image": { "src": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD...", "alt": "Blog Image Alt Text" },
  "content": "<p>This is the content of the blog post in HTML format.</p>"
}
```

**Event Object:**

| Data Type | Field | Min | Max | Required | Description |
|-----------|-------|-----|-----|----------|-------------|
| String | `title` | 5 | 100 | Yes | Title of the event. |
| String | `slug` | 5 | 150 | Yes | <u>Unique slug in events.</u> |
| String | `shortDesc` | 145 | 155 | Yes | Short description of the event. |
| String | `startDateTime` | 24 | 24 | Yes | ISO 8601 representation of the start date and time of the event. |
| String | `endDateTime` | 24 | 24 | Yes | ISO 8601 representation of the end date and time of the event. Should be later than `startDateTime`. |
| String | `location` | 15 | 255 | Yes | Location of the event. |
| String | `content` | 300 | 65,535 | Yes | Content describing the event in HTML format. |

Example:

```json
{
  "title": "Example Event Title",
  "slug": "example-event-title",
  "shortDesc": "This is a short description of the event.",
  "startDateTime": "2023-12-01T18:00:00Z",
  "endDateTime": "2023-12-01T20:00:00Z",
  "location": "Example Event Location",
  "content": "<p>This is the content describing the event in HTML format.</p>"
}
```

**Image Object:**

| Data Type | Field | Min | Max | Required | Description |
|-----------|-------|-----|-----|----------|-------------|
| String | `src` | - | 8MB | Yes | Image data specified as a data URI scheme, encoded in base64. |
| String | `alt` | 15 | 50 | Yes | Alternative text for the image. |

Example:

```json
{
  "src": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD...",
  "alt": "Image Alt Text"
}
```

**Authorization Object:**

| Data Type | Field | Required | Description |
|-----------|-------|----------|-------------|
| String | `email` | | |
| String | `password` | | |
| Unknown | \<bot validation fields> | Not currently defined. |

**Request Password Reset Object:**

| Data Type | Field | Required | Description |
|-----------|-------|----------|-------------|
| String | `email` | | |
| Unknown | \<bot validation fields> | Not currently defined. |

**Reset Password Object:**

| Data Type | Field | Required | Description |
|-----------|-------|----------|-------------|
| String | `password` | | |
| String | `retyped_password` | | |

## PATCH Body Objects

**User Object:**

| Data Type | Field | Min | Max | Nullable or Empty | Description |
|-----------|-------|-----|-----|-------------------|-------------|
| String | `firstName` | 2 | 50 | No | First name of the user. |
| String | `lastName` | 2 | 50 | Yes | Last name of the user. |
| String | `email` | - | 100 | No | <u>Unique email address associated with the user.</u> |

Example:

Let's say you want to update the firstName and email fields of a user:

```json
{
  "firstName": "NewFirstName",
  "email": "newemail@example.com"
}
```

**Blog Object:**

| Data Type | Field | Min | Max | Nullable or Empty | Description |
|-----------|-------|-----|-----|-------------------|-------------|
| String | `title` | 5 | 100 | No | Title of the blog. |
| String | `slug` | 5 | 150 | No | <u>Unique slug identifying the blog.</u> |
| String | `shortDesc` | 145 | 155 | No | Short description of the blog content. |
| Object | `image` | - | - | Yes | Unique identifier for the associated image. |
| String | `content` | 300 | 65,535 | No | Content of the blog in HTML format. |

Example:

If you wish to update the title and shortDesc fields of a blog:

```json
{
  "title": "New Blog Title",
  "shortDesc": "Updated short description of the blog."
}
```

**Event Object:**

| Data Type | Field | Min | Max | Null or Empty | Description |
|-----------|-------|-----|-----|---------------|-------------|
| String | `title` | 5 | 100 | No | Title of the event. |
| String | `slug` | 5 | 150 | No | <u>Unique slug identifying the event.</u> |
| String | `shortDesc` | 145 | 155 | No | Short description of the event. |
| String | `startDateTime` | 24 | 24 | No | ISO 8601 representation of the event's start date and time. |
| String | `endDateTime` | 24 | 24 | No | ISO 8601 representation of the event's end date and time. Must be later than `startDateTime`. |
| String | `location` | 15 | 255 | No | Location where the event will take place. |
| String | `content` | 300 | 65,535 | No | Content describing the event in HTML format. |

Example:

For modifying the title, startDateTime, and endDateTime fields of an event:

```json
{
  "title": "New Event Title",
  "startDateTime": "2023-12-01T18:00:00Z",
  "endDateTime": "2023-12-01T20:00:00Z"
}
```

**Image Object:**

| Data Type | Field | Min | Max | Null or Empty | Description |
|-----------|-------|-----|-----|---------------|-------------|
| String | `alt` | 15 | 50 | No | Alternative text description for the image. |

Example:

If you're updating the alt field of an image:

```json
{
  "alt": "New Image Alt Description"
}
```

## Referring Objects

### Link Object

The Link object is used to references an resource.

Structure: `<identifier>: { href: string, title: string, method: (POST|PATCH|DELETE) }`

| Data Type | Field | Description |
|-----------|-------|-------------|
| Object | `<identifier>` | Specifies an identification for the link. E.g., `self`, `update`, etc. [Predefined Identification] <br> Contains fields `href`, `title`, `method` |
| String | `href` | Absolute URL pointing to the referenced resource. |
| String | `title` | Descriptive title for the link. |
| String | `method` | (Optional) Specifies the HTTP method (`POST`, `PATCH`, `DELETE`) to be used when interacting with the link. |

Example:

```json
{
  "login": { 
    "href": "/login",
    "title": "Login",
    "method": "POST"
  }
}
```

**Predefined Identification:**

| Identification | Description | Example |
|----------------|-------------|---------|
| `self` | Reference to the resource itself. | `{ "self": { "href": "/users/0", "title": "Mads Hansen" } }` |
| `update` | Reference to update the resource. Can have a null value. | `{ "update": { "href": "/users/0", "title": "Update", "method": "PATCH" } }` |
| `delete` | Reference to delete the resource. Can have a null value. | `{ "delete": { "href": "/users/0", "title": "Delete", "method": "DELETE" } }` |
| `next` | Reference to the next page. Can have a null value. | `{ "next": { "href": "/users/?page=2", "title": "Next page"} }` |
| `login` | Reference to the login resource. | `{ "login": { "href": "/login", "title": "Login", "method": "POST"} }` |
| `reset` | Reference to request a password reset resource. | `{ "reset": { "href": "/reset_password", "title": "Reset password", "method": "POST" } }` |

### Image Object

| Data Type | Field | Description |
|-----------|-------|-------------|
| String | `src` | Image file name with its extension or specified as a data URI scheme, encoded in base64. |
| String | `alt` | Alternative text description for the image. |

Example:

```json
{
  "src": "example1.jpg",
  "alt": "Image Alt Text"
}
```

```json
{
  "src": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD...",
  "alt": "Image Alt Text"
}
```
