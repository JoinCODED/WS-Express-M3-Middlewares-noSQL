We will work on this Tourism Api, fork and clone [this](https://github.com/JoinCODED/Demo-Express-M3-Sql-Tourism) in your development folder.

At this point in our `monuments.controllers.js`, we have a lot of repeated code. Let's clean it up! To do that we need to learn about middleware methods.

1. In `app.js`, we will create a middleware method using the `app.use` method. Place it under the `body-parser` method. A middleware takes a callback function as an argument, and this method takes the request, response and something called `next` as arguments. Now this method will run every time anyone sends a request to our application, let's send a request to the monuments list method:

```javascript
app.use((req, res, next) => {
  console.log("I'm a middleware method");
});
```

2. So we got the message in the console, but we didn't receive a response! The reason is that we're stuck in the middleware method, to terminate it we will use `next`:

```javascript
app.use((req, res, next) => {
  console.log("I'm a middleware method");
  next();
});
```

3. Can we have more than one middleware method? Yes, the sky is your limit! Let's add another one **under** the first one:

```javascript
app.use((req, res, next) => {
  console.log("I'm another middleware method");
  next();
});
```

4. Another question. Is the place of the middleware important? Yes it is. Let's move the second middleware inside `monuments.routes` under the monuments list route. Now let's make a request to monuments list. We only got the first middleware message. Now let's make a request to the monuments create, we got both messages. So it is important!

```javascript
router.get('/', monumentsGet);

app.use((req, res, next) => {
  console.log("I'm another middleware method");
  next();
});

router.post('/', monumentsCreate);
```
