---
title: What are CORS? and how to configure it in Node?
date: 2024-11-12T16:38:52+08:00
updated: 2024-11-12T16:40:08+08:00
dg-publish: false
---

CORS, Cross-origin resource sharing, is a policy implemented by the browsers.

## Origin

First, let's understand what is origin. Origin is made up of the `base url` and the `port`. So `https://abc.com`, `https://abc.com:4040` and `https://cba.com` are different origins. Note that when we do not provide a port number with `<url>:<port>`, the default port for `https` is `443` and for `http`, its `80`. Hence, in the above example of `https://abc.com`, we have a port of `443` by default.

## Cross origin

Cross origin simply means that one origin is trying to connect to another origin.

## Cross origin resource sharing

For developing web applications, there are several architecture.

One of the ways to develop web application is to have the `Frontend` and `Backend` at the `same origin`, for example a NextJs Application, which is responsible for both, Frontend and Backend. In this case, whenever Frontend makes a network request to Backend, it requests on the `same origin`. And hence no CORS policy is implemented by browsers.

Other way to develop web application is to have `Frontend` and `Backend` at different origins. Let's say we got Frontend, React Application, at port `5173`, vite default port. And Backend, Node application, at port `3000`. In this case, we got different origins. And hence whenever a network request is made from React app to Node app, browser will implement CORS policy and possibly throwing an error

## Network Flow

Let's understand what happens, when a cross-origin network call is made.

1. Client side (Frontend) sends a `preflight` request to Server (Backend).
2. Server responds with whether the Client side's origin is allowed to request the Server or not. If server allows such origin, the `response headers` have the attribute `Access-Control-Allow-Origin` with value of origin / origin pattern.
3. Browser check the response headers and verify if its origin matches the origin pattern against the attribute of `Access-Control-Allow-Origin`. a. If it does matches, browser re-request the server with the actual payload and executes the required network call. b. If it does not matches, browser throws an error.

## Configuring backend to support CORS

Now we know that to configure CORS, we only need to modify our `response headers` to have the attribute of `Access-Control-Allow-Origin` which matches the origin pattern of the client.

One way of configuring CORS, is by simply using the package, `cors` and use the middleware provided by cors

```
const express = require("express");
const cors = require("cors");

const app = express();

app.use(cors({ origin: "http://localhost:5173" }));

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(3000, () => {
  console.log("Server is running on port 3000");
});
```

Now we got everything working, but there is another way of configuring response header, that is without using any package.

We simply need to define our custom middleware that modifies the response object. Here is how we can achieve it:

```
const express = require("express");

const app = express();

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "http://localhost:5173");
  next();
});

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(3000, () => {
  console.log("Server is running on port 3000");
});
```

**Note**: After the changes, you might not see the expected results, try using incognito or clear site cache.
