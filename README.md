# Edit, Update, and Delete Routes

## Lesson Objectives

1. Create a delete route
1. Make the index page send a DELETE request
1. Create an edit route
1. Create a link to the edit route
1. Create an update route
1. Make the edit page send a PUT request


## Setup

1. `cd ~/sei/express-fruits`
2. Open `express-fruits` in your editor.
3. `nodemon`

## Delete

### Create a delete route

Because we're just simulating a database so far, our delete route will need to take in an index and use it to `splice` out the value.

And, after data is deleted, what kind of response would we expect? We would likely be redirected somewhere. Since we can't be redirected back to the show page of a fruit that's no longer in the data, we should redirect to the index page where the user can see all their fruits, but without the recently-deleted item.

In `controllers/fruits_controllers.js`:

```js
// delete route
// this route will catch DELETE requests to /fruits/anyValue
// and, after deleting data, respond by redirecting
// the user to the index route
router.delete('/fruits/:fruitIndex', (req, res) => {
	 // remove the item from the array
    fruits.splice(req.params.fruitIndex, 1);
    // redirect back to index route
	res.redirect('/fruits');
});
```

### Make the index page send a DELETE request

Because we are using HTML only to send HTTP requests, and because HTML elements can only send GET or POST requests, we'll need to rely on extra tools to make a DELETE request successful.

By default, anchor tags (`<a>`) send GET requests, and there's no way to override this without involving JavaScript.

Form tags (`<form>`), however, typically send POST requests (though they can send GET requests as well), and we do have a way to override this.

Inside `views/index.ejs`, add a form with just a delete button.

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>Fruits index page</h1>
        <ul>
            <% for(let i = 0; i < allFruits.length; i++){ %>
                <li>
                    <a href="/fruits/<%= i %>">
                        <%= allFruits[i].name %>
                    </a>
                    <!--  ADD DELETE FORM HERE-->
                    <form action="/fruits/<%= i %>" method="POST">
                        <input type="submit" value="DELETE"/>
                    </form>
                </li>
            <% } %>
        </ul>
        <nav>
            <a href="/fruits/newForm">Create a New Fruit</a>
        </nav>
    </body>
</html>
```

If we click the DELETE button right now, which route will it it, if any?

We have no routes set up to listen for a POST request to `/fruits/:fruitIndex` so submitting this form will send a request that will not be heard. In fact, we'll get a message like `Cannot POST /fruits/0`, which indicates that the request URL that the client sent wasn't handled.

Of course, we could replace `POST` with `DELETE`, but the `form` tag doesn't know what DELETE is and will then assume this is a GET request.

How do we then make a DELETE request from a form? We'll need to kind of fake it with the help of an npm package called `method-override`.

1. Let's install it: `npm install method-override`
2. Let's import it in `server.js` toward the top:

```javascript
const methodOverride = require('method-override');
```

3. Now let's register it in our middleware above where our routes are registered:

```js
// registering methodOverride will allow us to add a query parameter called _method to our delete form
app.use(methodOverride('_method'));
```

When we call `methodOverride`, we pass it a string that we want to use as a query parameter key.

4. Back in `views/index.ejs`, let's modify our delete form `action`:

```html
<form action="/fruits/<%=i; %>?_method=DELETE" method="POST">
```

Our `methodOverride` middleware will listen for any requests that have a query parameter of `_method`. Then, it will take the value of that parameter (in this case, `DELETE`) and replace the value of `req.method` so it can be directed to the correct route. This is how a POST route can become a DELETE route.


## Update

### Create an edit route

When we want to edit data in an app, we click Edit (or click the pencil icon), we expect to have a form that's prefilled with the data that we want to modify.

If that's the case, how can we find that data in order to prefill the form? This route will need information (e.g., an id or a fruit index) to obtain the correct data.

In this case, we'd probably want to use a GET route that looks like this: `/fruits/:fruitIndex`. But that's already been taken by the show route. We'll have to make it unique, so let's add a third section to our URL path: `/fruits/:fruitIndex/editForm`

Now our route is unique and we still have a spot for the URL parameter. Notice that the parameter is still in the second spot. It doesn't actually matter which section(s) of the URL path are allotted for parameters. We just need to make sure that, when the browser sends the requests, the dynamic data is in that segment of the path.

We're going to need to make three changes: create an edit view, create a link to request that view (and make sure the fruit index is part of that request), and add a route that responds to that request by creating some HTML by rendering the view together with a fruit object.

1. Let's start with the route. In `controllers/fruits_controllers.js`, create a GET route which will just display an edit form for a single todo item:

```js
// edit route
// this route will catch GET requests to /fruits/anyValue/editForm
// and respond by rendering a form together with a fruit object
router.get('/fruits/:fruitIndex/editForm', (req, res) => {
	res.render('edit.ejs', {
        // the fruit object
        oneFruit: fruits[req.params.fruitIndex],
         // the index of the fruit object in the array
        index: req.params.fruitIndex
    });
});
```

Why do we need to send the `index` along with the fruit object? We have to think one step ahead. We need the fruit object to fill in the fields of the form. But, what is going to be the `action` of the form? When the form is submitted, what route is it going to trigger?

It will be triggering our next and final route &mdash; the update route &mdash; and that route will need to know the index of the fruit so it can find it in the `fruits` array and update its values.

By contrast, the new and create routes were simpler than the edit and update routes, right? This makes sense because the new and create routes just needed to work together to create something that didn't exist where the edit and update routes need to interact with existing data.

2. Now that the route has made data available to the view, let's create the view with `touch views/edit.ejs` and then populate with with a prefilled form:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>Edit Fruit Page</h1>
        <form>
            <!--  NOTE: the form is pre-populated with values for the server-->
            Name: <input type="text" name="name" value="<%= oneFruit.name %>"/><br/>
            Color: <input type="text" name="color" value="<%= oneFruit.color %>"/><br/>
            Is Ready To Eat:
            <input type="checkbox" name="readyToEat"
                <% if(oneFruit.readyToEat === true){ %>
                    checked
                <% } %>
            />
            <br/>
            <input type="submit" value="Submit Changes"/>
        </form>
    </body>
</html>
```

Notice that we used the `value` attribute to prefill the first two `input` fields.

For the checkbox, it's bit more complicated. When a checkbox is checked, a `checked` attribute is added. This is a boolean attribute, so if it's there, the box is checked and if it's not the box is empty. We want to conditionally add this attribute based on whether or not the `readyToEat` property of the fruit we're about to edit is true or false.

We'll come back and fill in the form's `action` and `method` later.

3. Now we need to create a link to get to this form. Let's do this in `views/index.ejs`:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>Fruits index page</h1>
        <ul>
            <% for(let i = 0; i < allFruits.length; i++){ %>
                <li>
                    <a href="/fruits/<%= i %>">
                        <%= allFruits[i].name %>
                    </a>
                    <form action="/fruits/<%= i %>" method="POST">
                        <input type="submit" value="DELETE"/>
                    </form>
                    <!-- Add edit link here  -->
                    <a href="/fruits/<%= i %>/editForm">Edit</a>
                </li>
            <% } %>
        </ul>
        <nav>
            <a href="/fruits/newForm">Create a New Fruit</a>
        </nav>
    </body>
</html>
```

Since the for loop gives us the index of each fruit, we can just use `i` as the parameter for our edit form link.

### Create an update route

Our final route will handle requests coming in when our edit form is submitted. It's HTTP verb is PUT.

In `controllers/fruits_controllers.js`:

```js
router.put('/fruits/:fruitIndex', (req, res) => {
    if(req.body.readyToEat === 'on'){
        req.body.readyToEat = true;
    } else {
        req.body.readyToEat = false;
    }
    // in our fruits array, find the index that is specified in the url (:fruitIndex) and overwrite the object with req.body (the form data)
	fruits[req.params.fruitIndex] = req.body;
	res.redirect(`/fruits/${req.params.fruitIndex}`);
});
```

Much like our create route, we need to convert the `readyToEat` value into a boolean before updating the fruit object.

Also, like our create route, what do we typically expect to happen after we've made the changes and submitted the form? Typically, we're redirected to an index of all our items where our newly-modified item will display with its updates among the other items. Or, we're redirected to a show page for that modified item. We've done the redirect to the index route a couple times, so this time we're redirecting to a show route.

Remember that `.redirect()` sends a GET request to the URL that's passed to it. Our show route is listening for a URL of `/fruits/:fruitIndex` so we just need to put a real index in that second segment of the URL. And, because `req.params.fruitIndex` contains the index we want, we can interpolate it into the URL string.

### Make the edit page send a PUT request

Now we need to go back and finish our edit form. Remember that the form can only send GET or POST requests, so a PUT request will require us to override the POST method. Like the delete route, we'll need to make use of the `method-override` package. Since we've already installed it, we don't have to do that again.

So, in `views/edit.ejs`:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>Edit Fruit Page</h1>
        <!-- Add action and method attributes to form tag -->
        <!-- Make sure the action uses the _method query parameter that method-override is looking for -->
        <form action="/fruits/<%= index %>?_method=PUT" method="POST">
            Name: <input type="text" name="name" value="<%= oneFruit.name %>"/><br/>
            Color: <input type="text" name="color" value="<%= oneFruit.color %>"/><br/>
            Is Ready To Eat:
            <input type="checkbox" name="readyToEat"
                <% if(oneFruit.readyToEat === true){ %>
                    checked
                <% } %>
            />
            <br/>
            <input type="submit" value="Submit Changes"/>
        </form>
    </body>
</html>
```

We've finished all the routes!