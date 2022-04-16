The code in the update and delete methods is very repeated, and the only difference is two lines. How can we clean them up?

1. Let's start with creating a function that does the fetching for us in `monuments.controllers`. This function takes the `monumentId` as an argument and returns the found monument:

   ```javascript
   exports.fetchMonument = async (monumentId) => {
     const monument = await Monument.findById(monumentId);
     return monument;
   };
   ```

2. Since we're dealing with a mongoose method, let's add the `try catch` statement as well.

   ```javascript
   exports.fetchMonument = async (monumentId) => {
     try {
       const monument = await Monument.findById(monumentId);
       return monument;
     } catch (error) {
       next(error);
     }
   };
   ```

3. In `monumentsUpdate`, let's replace `Monument.findById` with `fetchMonument`.

```javascript
exports.monumentsUpdate = async (req, res) => {
  const { monumentId } = req.params;
  try {
    const foundMonument = fetchMonument(+monumentId);
    if (foundMonument) {
      await foundMonument.findByIdAndUpdate(monumentId, req.body, {
        new: true,
      });
      res.status(204).end();
    }
  } catch (err) {
    const error = new Error('Monument Not Found');
    error.status = 404;
    next(error);
  }
};
```

4. But we'll get the error `next is not defined` if the request is failed. What to do?

5. We can easily pass `next` from `monumentsUpdate` to `fetchMonument` as a second argument! Viola! It's working perfectly.

   ```javascript
   exports.monumentsUpdate = async (req, res, next) => {
   const { monumentsId } = req.params;
   const monument = await fetchMonument(+monumentId, next);
   ```

   ```javascript
   exports.fetchMonument = async (monumentId, next) => {
     try {
       const monument = await Monument.findById(monumentId);
       return monument;
     } catch (error) {
       next(error);
     }
   };
   ```

6. Back to our update method, there's still a lot of repeated code.

7. To clean it up, in `monuments.routes` we will create a middleware that can take care of all methods that have the route parameter `monumentId`. This middleware is called `param`.

8. `param` has two arguments, the name of the route parameter that will trigger this middleware -which is `monumentId` in this case- and a callback function. The callback function has four arguments: the request, response, next function and the value saved in the route parameter. Let's console log something, to see what happens.

   ```javascript
   router.param('monumentId', async (req, res, next, monumentId) => {
     console.log(`The value of monumentId is ${monumentId}`);
   });
   ```

9. Let's make a request to the list and delete routes. As you can see this middleware method is **only** triggered when a route has `monumentId` as a route parameter.

10. What we'll do is call `fetchMonument` directly here and pass it `monumentId`. Then we will save the returned value inside the request object. Why? Because **all** routes have access to it!

```javascript
const {
  [...]
  fetchMonument,
} = require('./monuments.controllers');

router.param("monumentId", async (req, res, next, monumentId) => {
  const monument = await fetchMonument(+monumentId, next);
  req.monument = monument;
});
```

Since middleware has access to both the request and response objects, it can alter them or add data to them.

11. Let's test it! Oops, the request is stuck. Why is that? Remember that this is a middleware method, for it to end we need to use `next`.

```javascript
router.param('monumentId', async (req, res, next, monumentId) => {
  const monument = await fetchMonument(+monumentId, next);
  req.monument = monument;
  next();
});
```

12. But what if the monument does not exist. We can handle the not-found error here!

```javascript
router.param('monumentId', async (req, res, next, monumentId) => {
  const monument = await fetchMonument(+monumentId, next);
  if (monument) {
    req.monument = monument;
    next();
  } else {
    const err = new Error('Monument Not Found');
    err.status = 404;
    next(err);
  }
});
```

13. Now let's clean up `monumentsUpdate`. Whoa! How clean is this?! Unbelievable! `param` is amazing!

```javascript
exports.monumentsUpdate = async (req, res, next) => {
  try {
    await Monument.findByIdAndUpdate({ _id: req.monument.id });
    res.status(204).end();
  } catch (error) {
    next(error);
  }
};
```

14. Let's clean up `monumentsDelete` as well!

```javascript
exports.monumentsDelete = async (req, res, next) => {
  try {
    await Monument.findByIdAndRemove({ _id: req.monument.id });
    res.status(204).end();
  } catch (err) {
    next(error);
  }
};
```
