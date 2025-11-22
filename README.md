# 25 Backend Interview Questions (Beginner to Intermediate)
Based on `hcltech_healthcare` Project

## General Node.js & Express

### 1. What is the purpose of `package.json` in this project?
**Answer:** It is the manifest file for the Node.js project. It contains metadata about the project (name, version), lists dependencies (like `express`, `mongoose`), defines scripts (like `start`, `dev`, `test`), and configures project settings.

### 2. What is the difference between `dependencies` and `devDependencies`?
**Answer:** `dependencies` (e.g., `express`, `mongoose`) are required for the application to run in production. `devDependencies` (e.g., `nodemon`, `jest`) are only needed for local development and testing purposes.

### 3. Explain the role of `server.js` vs `app.js` in this codebase.
**Answer:** 
- `app.js` sets up the Express application, middleware (cors, helmet), and routes. It exports the `app` instance.
- `server.js` imports the `app`, connects to the database, and starts the server listening on a specific port. This separation makes testing easier (you can test `app` without starting the server).

### 4. What does `app.use(express.json())` do in `app.js`?
**Answer:** It is a built-in middleware function in Express. It parses incoming requests with JSON payloads (headers `Content-Type: application/json`) and puts the parsed data in `req.body`. Without this, `req.body` would be undefined for JSON requests.

### 5. What is Middleware in Express? Give an example from the project.
**Answer:** Middleware functions have access to the request object (`req`), the response object (`res`), and the next middleware function (`next`). They can execute code, modify req/res, or end the cycle. 
*Example:* The `protect` function in `authMiddleware.js` is middleware that verifies the JWT token before allowing access to protected routes.

### 6. What is the purpose of `cors` and why is it used in `app.js`?
**Answer:** CORS (Cross-Origin Resource Sharing) is a security feature in browsers. The `cors` middleware allows the backend to accept requests from different origins (domains). In `app.js`, `app.use(cors({ origin: '*' }))` allows *any* domain to access the API (useful for development, but should be restricted in production).

### 7. What does the `helmet` middleware do?
**Answer:** `helmet` helps secure Express apps by setting various HTTP headers. It protects against common web vulnerabilities like cross-site scripting (XSS) and clickjacking.

### 8. What is `morgan` used for?
**Answer:** `morgan` is an HTTP request logger middleware. In `app.js`, `app.use(morgan('dev'))` logs details about incoming requests (method, URL, status code, response time) to the console, which helps with debugging.

### 9. How does the global error handler work in `app.js`?
**Answer:** It is defined as `app.use(errorHandler)` at the end of the middleware stack. If any route throws an error or calls `next(error)`, Express passes execution to this error handling middleware, which formats the error response sent to the client.

### 10. What is `dotenv` and why do we use `.env` files?
**Answer:** `dotenv` loads environment variables from a `.env` file into `process.env`. This is used to keep sensitive configuration data (like `MONGO_URI`, `JWT_SECRET`) out of the codebase and version control.

---

## Database (MongoDB & Mongoose)

### 11. What is Mongoose?
**Answer:** Mongoose is an Object Data Modeling (ODM) library for MongoDB and Node.js. It provides a schema-based solution to model application data, offering features like type casting, validation, query building, and business logic hooks.

### 12. Explain the purpose of `mongoose.Schema` in `User.js`.
**Answer:** It defines the structure of the documents within a MongoDB collection. It specifies the fields (name, email, password), their types (String, Number), and validation rules (required, unique).

### 13. What does the `timestamps: true` option do in a Mongoose schema?
**Answer:** It automatically adds two fields to the schema: `createdAt` (date when the document was created) and `updatedAt` (date when the document was last modified). Mongoose manages these fields automatically.

### 14. In `WellnessData.js`, what does `ref: 'User'` signify?
**Answer:** It establishes a relationship between the `WellnessData` model and the `User` model. It tells Mongoose that the `user` field contains an `ObjectId` that references a document in the `users` collection, allowing for population (joins) later.

### 15. What is the purpose of the `unique: true` property in the User email field?
**Answer:** It creates a unique index in MongoDB for the email field, ensuring that no two users can register with the same email address.

---

## Authentication & Security

### 16. What is JWT (JSON Web Token) and how is it used here?
**Answer:** JWT is a compact, URL-safe means of representing claims to be transferred between two parties. In this project, when a user logs in, the server generates a JWT signed with a secret. The client sends this token in the `Authorization` header for subsequent requests to authenticate themselves.

### 17. Why do we hash passwords using `bcrypt` before saving them?
**Answer:** Storing passwords in plain text is a major security risk. If the database is compromised, attackers would have all user passwords. `bcrypt` hashes the password (scrambles it irreversibly) so that even if the data is stolen, the actual passwords remain secure.

### 18. Explain the `pre('save')` hook in `User.js`.
**Answer:** This is a Mongoose middleware that runs *before* a document is saved to the database. In `User.js`, it checks if the password field has been modified. If so, it hashes the new password using `bcrypt` before saving the user document.

### 19. What is a "Salt" in the context of `bcrypt`?
**Answer:** A salt is random data added to the password before hashing. It ensures that even if two users have the same password, their resulting hashes will be different, protecting against rainbow table attacks. `bcrypt.genSalt(10)` generates this salt.

### 20. How does the `protect` middleware know which user is making the request?
**Answer:** It extracts the JWT token from the request headers, verifies it using the `JWT_SECRET`, and decodes the payload (which contains the user ID). It then fetches the user from the database and attaches it to `req.user`, making the user data available to the route controller.

---

## API Design & Architecture

### 21. What is the difference between `PUT` and `PATCH`? (Hint: `wellnessRoutes.js` uses PATCH)
**Answer:** 
- `PUT` is used to replace an entire resource or document.
- `PATCH` is used to apply partial modifications to a resource. In `wellnessRoutes.js`, `PATCH /:metric` is used because we only want to update a specific part of the wellness data (e.g., just the steps count) without sending the entire object.

### 22. Why do we separate `routes` and `controllers`?
**Answer:** This follows the Separation of Concerns principle. 
- **Routes** (`routes/`) define the API endpoints (URLs) and map them to specific functions.
- **Controllers** (`controllers/`) contain the actual business logic and request handling code. 
This makes the code cleaner, more readable, and easier to maintain.

### 23. What does `res.status(201)` mean vs `res.status(200)`?
**Answer:** 
- `200 OK`: The request succeeded (standard response for successful GET, PUT, PATCH).
- `201 Created`: The request succeeded and a new resource was created (standard response for a successful POST, like User Registration).

### 24. Why are we using `async/await` in the controllers?
**Answer:** Database operations (like `User.findOne` or `user.save`) are asynchronous and return Promises. `async/await` allows us to write asynchronous code that looks and behaves like synchronous code, making it easier to read and handle errors compared to using `.then()` and `.catch()` chains.

### 25. What is the purpose of `process.env.PORT || 8080` in `server.js`?
**Answer:** It sets the port the server listens on. It tries to use the `PORT` environment variable (commonly provided by hosting platforms like Heroku or Google Cloud Run). If that variable is not set (e.g., running locally without a .env specifying it), it defaults to port `8080`.
