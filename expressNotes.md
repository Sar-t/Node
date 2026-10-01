# CORS: Cross-Origin Resource Sharing
It means web request is made from different origin. 
It is because backend listens to only its origin request.
So for this wee use proxies but only in deployment. Using proxy following happens
 - We can omit the domain name in every fetch api call. So we just have to write the endpoint name
 ```js
 fetch(`/api/home`)
 ```
 The domain name will be appened automatically.
 - Server identifies this web request as of same origin(port) as itself.

```js
"/api":{
    target: "http://localhost:3000",
}
//This will match every web request if it start with /api it will tell react server to not handle it itself bu to send it to target server. So the request will be http://localhost:3000/api/joke for fetch(`/api/joke`)
```

In production we can use cors at backend to white list our frontend origin. Or use tools like nginx for reverse proxy.


# Data modeling with mongoose
## MongoDB:
It is a NOSQL document database that stores data in json like documents(binary json technically).
Instead of using tables and rows like SQL databases, MongoDB uses:
 - Database → contains collections
 - Collection → similar to a table
 - Document → similar to a row, stored as a JSON-like object

## Mongoose
Mongoose is a Node.js library (ODM – Object Data Modeling) that makes it easier to work with MongoDB.

Think of it as a layer between your Node.js application and MongoDB.

It provides features such as:

Schemas
Data validation
Models
Middleware (hooks)
Query helpers
Relationships between documents


```js
import mongoose from "mongoose"

const UserSchema = new mongoose.Schema(
  {
    username: String,
    email: String,
    isActive: Boolean
  }
);

export const User = mongoose.model("User",userSchema);
```

```js
import mongoose from 'mongoose';

const UserSchema = new mongoose.Schema(
  {
    username: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
    },
    password: {
      type: String,
      required: [true, 'Password is required!'],
    },
    isActive: Boolean,
  },
  { timestamps: true }
);

export const User = mongoose.model('User', userSchema);
```

```
Schema
   ↓
mongoose.model()
   ↓
User Model
   ↓
MongoDB collection
```

The model gives you methods to interact with the database:
```
User.create(...)
User.find(...)
User.findOne(...)
User.findById(...)
User.findByIdAndUpdate(...)
User.deleteOne(...)
```



### Referencing another model
```js
import mongoose from "mongoose"

const todoSchema = new mongoose.Schema({
  content:{
    type: String,
    required: true,
  },
  complete:{
    type: Boolean,
    default: false,
  },
  createdBy:{
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
    
  }
},{timestamps:true});

export const Todo = mongoose.model("Todo",todoSchema);
```

The model name in reference should be same as defined in User model.
```js
export const User = mongoose.model(*'User'*, userSchema); //inside mongoose.model not export name
```

### Array of objects:
```js
subTodo:[
    {
      type:  mongoose.Schema.Types.ObjectId,
      ref: "SubTodo"
    },
    
  ]
```


**Note: Use try-catch  whenever use async await. For this we can we create a seperate utility wrapper.**


## Controller
callback or business logic
parameters: (err,req,res,next)

**Node.js provides us with an *`Error class`* which we can use for controlling the error response.**

## Status Codes

|    Code | Name                  | Meaning                                                 | Common Use                  |
| ------: | --------------------- | ------------------------------------------------------- | --------------------------- |
| **200** | OK                    | Request succeeded                                       | Successful GET, PUT, PATCH  |
| **201** | Created               | Resource successfully created                           | Successful POST             |
| **202** | Accepted              | Request accepted for processing                         | Async/background processing |
| **204** | No Content            | Request succeeded, but no response body                 | Successful DELETE           |
| **400** | Bad Request           | Request is invalid                                      | Invalid/missing input       |
| **401** | Unauthorized          | Authentication is required/failed                       | Invalid or missing token    |
| **403** | Forbidden             | User is authenticated but not allowed                   | Insufficient permissions    |
| **404** | Not Found             | Resource doesn't exist                                  | User/record not found       |
| **405** | Method Not Allowed    | HTTP method isn't supported                             | POST on a GET-only route    |
| **409** | Conflict              | Request conflicts with existing state                   | Duplicate email/username    |
| **422** | Unprocessable Content | Input format is valid but data is semantically invalid  | Validation failure          |
| **429** | Too Many Requests     | Too many requests in a given time                       | Rate limiting               |
| **500** | Internal Server Error | Unexpected server-side error                            | Unhandled exception         |
| **501** | Not Implemented       | Server doesn't support the requested functionality      | Unsupported feature         |
| **502** | Bad Gateway           | Server received an invalid response from another server | AI service/API failure      |
| **503** | Service Unavailable   | Server temporarily unavailable                          | Database/service down       |
| **504** | Gateway Timeout       | Upstream service didn't respond in time                 | AI/API timeout              |


**Node.js and express are best for I/O heavy tasks like in chat room or real time applications. There it is preferred.**

## Mongoose Paginate v2:
mongoose-paginate-v2 is a Mongoose plugin that makes normal MongoDB queries easier to paginate.

# Authentication

## Authentication vs Authorization

- **Authentication** → Verifies who the user is.
- **Authorization** → Determines what the user is allowed to do.

---

## Bcrypt

**Bcrypt** is used to securely hash passwords before storing them in the database.

### Hash Password

```js
const hashedPassword = await bcrypt.hash(password, 10);
Verify Password
const isValid = await bcrypt.compare(password, hashedPassword);
```

Bcrypt is one-way hashing. Passwords cannot be decrypted from the hash.

### JWT
Bearer-token. Anybody having this is authentic.
JWT (JSON Web Token) is used to authenticate users after login.

A JWT is signed, allowing the server to verify that it hasn't been modified.

Access Token
Short-lived token.
Used to access protected APIs.
Example lifetime: 15 minutes.

```
Login
  ↓
Access Token
  ↓
Protected API Requests
Refresh Token
Long-lived token.
Used to generate a new access token when the old one expires.
Commonly stored in a HttpOnly and Secure cookie.
Access Token expires
        ↓
Refresh Token
        ↓
New Access Token
Authentication Flow
Registration
User Password
      ↓
bcrypt.hash()
      ↓
Hashed Password
      ↓
Database
Login
Email + Password
       ↓
bcrypt.compare()
       ↓
Valid Credentials
       ↓
Generate Access + Refresh Token
Using the Application
Access Token
     ↓
API Request
     ↓
Server verifies JWT
     ↓
Protected Resource

When the access token expires:

Access Token expires
       ↓
Refresh Token
       ↓
Generate New Access Token
       ↓
Continue API Requests
```
```js
userSchema.pre("save",async function(next){
    if(!this.isModified("password")) return next();
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
});
```
pre is a hook and also a part of middleware chain.
 - this points to the document being saved.
 - we dont use callbacks as they dont have access to this.

So we define middleware using pre and post hook. These are automatically triggered once the event mentioned happens.

**We can create methods in our schema like this. They need to be explicitly called!**

```js
userSchema.methods.isPasswordCorrect = async function(password){
    return await bcrypt.compare(password, this.password);
}
```

## HTTP
HTTP (HyperText Transfer Protocol) is a set of rules that defines how a client and server communicate over a network.

URL - Locator
URI - Identifier
URN - Name

Headers: sent along with req and res
meta-data; key-value pair; 
 - Accept header: application/json
 - user-agent header: tells the source of request, brwoser or postman
 - authorization: 
 - content-type: images,pdf etc.
 - cookie: key value pairs.
 - cacche-control: expire cache limit
CORS headers:
 - access-control-allow-origin: 
 - access-control-allow-credentials - cookies generally
 - access-control-allow-method

| Method      | Purpose                                | Example                |
| ----------- | -------------------------------------- | ---------------------- |
| **GET**     | Retrieve data                          | `GET /api/firs`        |
| **POST**    | POST submits data to a server for processing, often resulting in the creation of a resource or triggering an action.                  | `POST /api/firs`       |
| **PUT**     | Completely replace/update a resource   | `PUT /api/firs/123`    |
| **PATCH**   | Partially update a resource            | `PATCH /api/firs/123`  |
| **DELETE**  | Delete a resource                      | `DELETE /api/firs/123` |
| **HEAD**    | Same as GET but returns headers only. To see user-agent,cache control.   | `HEAD /api/firs`       |
| **OPTIONS** | Ask what methods/options are supported | `OPTIONS /api/firs`    |

| Range   | Category      | Meaning                                |
| ------- | ------------- | -------------------------------------- |
| **1xx** | Informational | Request received, processing continues |
| **2xx** | Success       | Request was successfully processed     |
| **3xx** | Redirection   | Further action is needed               |
| **4xx** | Client Error  | Problem with the client's request      |
| **5xx** | Server Error  | Problem on the server                  |

## Express Response: `return`

`return` is **not required** when sending a response.

`res.status()` only sets the HTTP status code, while `.json()`, `.send()`, etc. actually send the response.

Use `return` when you want to **stop further execution** of the current function, especially after sending an error response.

```js
const registerUser = asyncHandler(async (req, res) => {

    if (!req.body.email) {
        return res.status(400).json({
            success: false,
            message: "Email is required"
        });
    }

    // This won't execute if email is missing
    const user = await createUser(req.body);

    res.status(201).json({
        success: true,
        user
    });
});
```

## Express Router

`app.get()` / `app.post()` can directly define routes, but for larger applications we use `express.Router()` to organize routes into separate modules.

`app.use("/api/firs", firRouter)` which is a middleware mounts the FIR router at `/api/firs`.

The router then handles routes such as:
- `GET /api/firs`
- `POST /api/firs`
- `GET /api/firs/:id`

### Why use Router?

- Keeps `app.js` clean
- Separates routes by feature/resource
- Improves modularity and maintainability
- Makes large applications easier to manage

> `app.use()` mounts a router; `router.get()`, `router.post()`, etc. define the actual routes inside that router.

## HTTPS
HTTPS (Hypertext Transfer Transfer Protocol Secure) secures the connection between a user's web browser and your website's server.


### Three Main Protections HTTPS Provides
 - Encryption: Scrambles data sent between the user and server, hiding passwords, credit card numbers, and personal info from hackers. 
    So when we send data using http no encryption is there plain text is sent instead.
 - Authentication: Proves your website is legitimate, ensuring users are connecting to your actual server and not a fake lookalike site. 

 - Data Integrity: Prevents third parties from altering your website's data mid-transit, blocking attackers from injecting malicious ads or malware into your pages

# Express Request–Response Flow

When a client (React/Browser) sends an HTTP request, Express processes it through middleware, routers, and controllers before sending a response.

## Flow

Client (React/Browser)
        ↓
HTTP Request
        ↓
Node.js HTTP Server
        ↓
Express
        ↓
Global Middleware
(express.json(), cors(), etc.)
        ↓
Router
        ↓
Route
        ↓
Authentication Middleware (if required)
        ↓
Controller
        ↓
Database
        ↓
Response (res.status().json())
        ↓
HTTP Response
        ↓
Client

## 1. Client Sends Request

The browser sends an HTTP request containing:

- HTTP method
- URL
- Headers
- Request body (if applicable)

## 2. Node.js Receives Request

The operating system sends the request to the Node.js process listening on the specified port.

## 3. Express Processes Request

Express creates:

- `req` → Request object
- `res` → Response object

Express then starts processing the request through middleware in the order they were registered.

## 4. Middleware

Middleware can:

- Parse request data (`express.json()`)
- Handle CORS
- Authenticate users
- Modify `req` or `res`
- End the request
- Pass control to the next middleware using `next()`

## 5. Router and Route

`app.use()` can mount a router at a specific path.

For example, `/api/auth` can be handled by an authentication router.

The router then finds the matching HTTP method and route, such as:

`POST /register`

## 6. Controller

The controller contains the business logic.

It may:

- Validate input
- Interact with the database
- Create/update data
- Generate tokens
- Send the response

## 7. Database

The controller may communicate with MongoDB through Mongoose to read or modify data.

## 8. Response

The controller sends the response using methods such as:

- `res.json()`
- `res.send()`
- `res.status().json()`

The response then travels back to the client.

# CORS

**CORS (Cross-Origin Resource Sharing)** is a browser security mechanism that controls whether a frontend from one origin can access a backend from another origin.

Example:

Frontend: `http://localhost:5173`
Backend: `http://localhost:3000`

The server sends CORS headers such as:

`Access-Control-Allow-Origin`

These headers tell the browser which origins are allowed.

## CORS Preflight

For some cross-origin requests, the browser first sends an `OPTIONS` request called a **preflight request**.

The flow is:

Browser → OPTIONS → Server

Server → CORS permission headers → Browser

Browser → Actual Request → Server

Server → Response → Browser

> CORS is mainly enforced by the browser. Express/CORS middleware provides the headers that tell the browser what cross-origin communication is allowed.

