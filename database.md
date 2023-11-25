# Database Documentation

## User Table
| Field       | Type                  | Required? | Nullable? | Description                               |
|-------------|-----------------------|-----------|-----------|-------------------------------------------|
| uid         | Integer (Auto Increment) | Yes       | No        | Unique identifier for the user.           |
| firstName   | Varchar(50)           | Yes       | No        | First name of the user.                   |
| lastName    | Varchar(50)           | No        | Yes       | Last name of the user.                    |
| email       | Varchar(100) UNIQUE   | Yes       | No        | Unique email address for the user.        |
| password    | Varchar(100)          | No        | Yes       | Hashed password for the user.             |
| createdAt   | DateTime              | Yes       | No        | Date and time of user creation.           |
| updatedAt   | DateTime              | No        | Yes       | Date and time of last update for the user.|
| loggedInAt  | DateTime              | No        | Yes       | Date and time of last login for the user. |

**Notes**:
- The `password` field supports nullable values for users created without an initial password and requires resetting before login.

## Blog Table
| Field       | Type                  | Required? | Nullable? | Description                               |
|-------------|-----------------------|-----------|-----------|-------------------------------------------|
| uid         | Integer (Auto Increment) | Yes       | No        | Unique identifier for the blog.           |
| title       | Varchar(100)          | Yes       | No        | Title of the blog.                        |
| slug        | Varchar(150) UNIQUE   | Yes       | No        | Unique URL-formatted string for the blog. |
| shortDesc   | Varchar(155)          | Yes       | No        | Brief description of the blog.            |
| content     | Text                  | Yes       | No        | Content of the blog in HTML format.       |
| createdAt   | DateTime              | Yes       | No        | Date and time of blog creation.           |
| updatedAt   | DateTime              | No        | Yes       | Date and time of last update for the blog.|

## Blog Relation Image Table
| Field       | Type                   | Required? | Nullable? | Description                                           |
|-------------|------------------------|-----------|-----------|-------------------------------------------------------|
| blogID      | Foreign Key (blog.uid) | Yes       | No        | Reference to the associated blog in the blog table.   |
| imageID     | Foreign Key (image.uid)| Yes       | No        | Reference to the associated image in the image table. |

**Relation**: Many-to-One relation between Blog and Image tables. Each entry in this table associates a single blog with a single image, allowing multiple blogs to share the same image.

**Constraint**: The combination of `blogID` and `imageID` must form a unique pair.

## Event Table
| Field         | Type                  | Required? | Nullable? | Description                                 |
|---------------|-----------------------|-----------|-----------|---------------------------------------------|
| uid           | Integer (Auto Increment) | Yes       | No        | Unique identifier for the event.            |
| title         | Varchar(100)          | Yes       | No        | Title of the event.                         |
| slug          | Varchar(150) UNIQUE   | Yes       | No        | Unique URL-formatted string for the event.  |
| shortDesc     | Varchar(155)          | Yes       | No        | Brief description of the event.             |
| startDateTime | DateTime              | Yes       | No        | Date and time of event start.               |
| endDateTime   | DateTime              | Yes       | No        | Date and time of event end.                 |
| location      | Varchar(255?)         | Yes       | No        | Location of the event.                      |
| content       | Text                  | Yes       | No        | Content of the event in HTML format.        |
| createdAt     | DateTime              | Yes       | No        | Date and time of event creation.            |
| updatedAt     | DateTime              | No        | Yes       | Date and time of last update for the event. |

## Image Table
| Field         | Type                  | Required? | Nullable? | Description                               |
|---------------|-----------------------|-----------|-----------|-------------------------------------------|
| uid           | Integer (Auto Increment) | Yes       | No        | Unique identifier for the image.          |
| src           | Varchar(100) UNIQUE   | Yes       | No        | Filename: `<image name>.ext`.             |
| alt           | Varchar(50)           | Yes       | No        | Alternate text for the image.             |
| createdAt     | DateTime              | Yes       | No        | Date and time of image creation.          |
| updatedAt     | DateTime              | No        | Yes       | Date and time of last update for the image. |

## Access Token Table
| Field       | Type                  | Required? | Nullable? | Description                                       |
|-------------|-----------------------|-----------|-----------|---------------------------------------------------|
| userID      | Foreign Key (user.uid)| Yes       | No        | Reference to the unique identifier in User table. |
| token       | Varchar(255?)         | Yes       | No        | Authorization access token.                       |
| ip          | Varchar(45)           | Yes       | No        | IP address associated with the token.             |
| expiresAt   | Timestamp             | Yes       | No        | Expiry timestamp for the token.                   |

**Relation**: One-to-Many (One user can have many access tokens)

**Constraint**: The combination of `userID`, `token`, and `ip` must form a unique set, ensuring that each token associated with a user is tied to a specific IP address.

## Bot Protection Table
| Field       | Type                  | Required? | Nullable? | Description                                              |
|-------------|-----------------------|-----------|-----------|----------------------------------------------------------|
| token       | Varchar(255?)         | Yes       | No        | Token stored in a cookie to remember the user.           |
| userValue   | Varchar(50)           | Yes       | No        | The value that a user needs to provide to prove non-bot. |
| expiresAt   | Timestamp             | Yes       | No        | Expiry timestamp for the token.                          |

## Reset Token Table
| Field       | Type                  | Required? | Nullable? | Description                                       |
|-------------|-----------------------|-----------|-----------|---------------------------------------------------|
| userID      | Foreign Key (user.uid)| Yes       | No        | Reference to the unique identifier in User table. |
| token       | Varchar(255?)         | Yes       | No        | The value sent to the user as a link.             |
| expiresAt   | Timestamp             | Yes       | No        | Expiry timestamp for the token.                   |

**Constraint**: Only one active reset token is allowed per user.
