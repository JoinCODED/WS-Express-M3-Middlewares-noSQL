A common use for middleware is handling requests to paths that don't exist. Let's try making a request to any path, for example `/lailz`. We get a `404` status code, but no message. So we'll create a middleware that catches requests for non-existing paths.

The correct place for this middleware is right before the `run()` method in `app.js`. This way, it only runs if the request doesn't match with any of our routes. If the request matches with a router, the route will return a JSON response, then this middleware won't be run, which is what we want.

1. In this middleware, all we want is to do is set the status code to `404` which means `Not Found` and send a message. To do that we will end the request with the following response:

```javascript
app.use((req, res, next) => {
  res.status(404).json({ message: 'Path not found' });
});
```

2. As you can see, a middleware can be ended using `next` or by sending back a response!

3. Let's try the path `/lailz` again. The middleware method and message are being sent back!
