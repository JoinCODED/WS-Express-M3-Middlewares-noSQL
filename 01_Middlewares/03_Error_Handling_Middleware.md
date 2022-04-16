1. Middleware methods are well-known for handling all types of errors. Actually, there is a special type of middleware that's specified for handling errors. What differentiates it from a normal middleware is that it has an extra argument at the beginning, which is the error object. For now let's log a message and the error object `err`.

```javascript
app.use((err, req, res, next) => {
  console.log("I'm an error handling middleware ", err);
});
```

2. To call an error middleware, you pass `next()` an object, which is the error object. When you do that, Express will directly go to the next middleware function with four arguments. Let's try it out in our `monumentsCreate` method by adding it in our `catch` block.

```javascript
exports.monumentsCreate = async (req, res) => {
  try {
    const newMonument = await Monument.create(req.body);
    res.status(201).json(newMonument);
  } catch (error) {
    next(error);
  }
};
```

1. We will send a request to Postman without the `name` -which is required- to get an error. Note that the message from the error middleware and the error object were logged.

2. Now what we want from this middleware is to handle the errors by terminating the request with a status code and a message that explains the error.

```javascript
app.use((err, req, res, next) => {
  res.status(err.status);
  res.json({ message: err.message });
});
```

5. But what if the passed error object didn't have a message or a status code? We need to set a default, generic value for both. The default status code will be `500` which basically means `Internal Server Error`.

```javascript
app.use((err, req, res, next) => {
  res.status(err.status || 500);
  res.json({
    message: err.message || 'Internal Server Error',
  });
});
```

6. Now we'll go to all our controller methods and replace every `catch` block content with a call to the error middleware using `next`.

```javascript
catch (error) {
  next(error);
}
```

7. Also we will leave the error handling in the update and delete methods when no Monument is found to the error middleware. So we will create an error instance using the JavaScript `Error` class change the status code to `404`.

```javascript
catch (err){
  const error = new Error("Monument Not Found");
  error.status = 404;
  next(error);
}
```
