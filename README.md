## FIRST EXERCISE

- create account on MongoDB.com
- create first cluster
- create a DB user
- Whitelist all IP addresses
- create MongoDB url

# Intro to Mongoose

## Lesson Objectives

1. Explain what an ODM is
1. Connect to Mongo via text editor
1. Create a Schema for a collection
1. Create a model and save it
1. find a specific model
1. update a model already in the database
1. remove a model already in the database
1. combine actions

## Explain what is an ODM/ Intro to Mongoose

ODM stand for Object Document Model. It translates the documents in Mongo into upgraded JavaScript Objects that have more helpful methods and properties when used in conjunction with express.

Rather than use the Mongo shell to create, read, update and delete documents, we'll use an npm package called `mongoose`. Mongoose will allow us to create schemas, do validations and make it easier to interact with Mongo inside an express app.

![Mongoose Visual](Client_Server.png)

## Make a Schema

A schema will allow us to set specific keys in our objects. So if we have a key of `name`, we won't be able to insert other keys that don't match like `firstName` or `names`. This helps keep our data more organized and reduces the chance of errors.

We can also specify the datatypes. We can set the datatype of `name` to a `string`, `age` to a `number`, `dateOfBirth` to a Date, `bff` to a Boolean etc.

We can also make some fields required and we can set default values as well.

Here is a sample Schema, with many options. We'll be making a smaller variation of this

```js
const articleSchema = new Schema(
  {
    title: { type: String, required: true, unique: true }, //can say whether we want properties to be required or unique
    author: { type: String, required: true },
    body: String,
    comments: [{ body: String, commentDate: Date }], // can have arrays of objects with specific properties
    publishDate: { type: Date, default: Date.now }, // can set defaults for properties
    hidden: Boolean,
    meta: {
      // can have properties that are objects
      votes: Number,
      favs: Number,
    },
  },
  { timestamps: true }
);
```

## Basic Set Up

In `student_examples`

- `mkdir intro_to_mongoose`
- `cd intro_to_mongoose`
- `touch app.js`
- `npm init -y` and go through the prompts
- `npm i mongoose`
- `touch tweet.js`
- `code .`

## Set Up Mongoose

Inside `app.js`

- require mongoose

```js
// Dependencies
const mongoose = require("mongoose");
const Tweet = require("./tweet.js");
```

- tell Mongoose where to connect with Mongo and have it connect with the sub-database `tweets` (if it doesn't exist, it will be created)
- set `mongoose.connection` to a shorter variable name

```js
// Global configuration
const mongoURI = "YOUR MONGODB URL";
const db = mongoose.connection;
```

- Connect to mongo

```js
// Connect to Mongo
mongoose.connect(mongoURI);
```

Getting a warning like this?
![depreciation](https://i.imgur.com/47eb1oo.png)

Warnings are ok, it'll still work, for now. But in later versions it may stop working and you'll have to update your code.

This should clear up the errors:

```js
mongoose.connect(mongoURI, { useNewUrlParser: true, useUnifiedTopology: true });
```

- **OPTIONAL** provide error/success messages about the connections

```js
// Connection Error/Success
// Define callback functions for various events
db.on("error", (err) => console.log(err.message + " is mongod not running?"));
db.on("open", () => console.log("mongo connected: ", mongoURI));
db.on("close", () => console.log("mongo disconnected"));
```

- While the connection is open, we won't have control of our terminal. If we want to regain control, we have to close the connection.
  Let's set leave the connection open for 5 seconds to demonstrate that the app will hang and then we'll get our close message.

Otherwise we have to press `control c`. When we run an express app, we typically want to leave the connection open, we don't need to get control of terminal back, we just let the app run.

```js
// Automatically close after 5 seconds
// for demonstration purposes to see that you must use `db.close()` in order to regain control of Terminal tab
setTimeout(() => {
  db.close();
}, 5000);
```

- The entire configuration for mongoose:
- Don't memorize it, just set a bookmark and refer back to this as you need it.
- note the setTimeout was just to demonstrate what `db.close()` does, you don't always need it

```js
// Dependencies
const mongoose = require("mongoose");
const Tweet = require("./tweet.js");

// Global Configuration
const mongoURI = "mongodb://localhost:27017/" + "tweets";
const db = mongoose.connection;

// Connect to Mongo
mongoose.connect(mongoURI, { useNewUrlParser: true, useUnifiedTopology: true });

// Connection Error/Success - optional but can be helpful
// Define callback functions for various events
db.on("error", (err) => console.log(err.message + " is Mongod not running?"));
db.on("open", () => console.log("mongo connected: ", mongoURI));
db.on("close", () => console.log("mongo disconnected"));
```

## Set Up Tweet Schema

In `tweet.js`

```js
const mongoose = require("mongoose"); // require mongoose
const Schema = mongoose.Schema; // create a shorthand for the mongoose Schema constructor
const model = mongoose.model // shorthand for model function

// create a new Schema
// This will define the shape of the documents in the collection
// https://mongoosejs.com/docs/guide.html
const tweetSchema = new Schema(
  {
    title: String,
    body: String,
    author: String,
    likes: { type: Number, default: 0 },
    sponsored: { type: Boolean, default: false },
  },
  { timestamps: true }
);

// Creating Tweet model : We need to convert our schema into a model-- will be stored in 'tweets' collection.  Mongo does this for you automatically
// Model's are fancy constructors compiled from Schema definitions
// An instance of a model is called a document.
// Models are responsible for creating and reading documents from the underlying MongoDB Database
// from here: https://mongoosejs.com/docs/models.html
const Tweet = model("Tweet", tweetSchema);

//make this exportable to be accessed in `app.js`
module.exports = Tweet;
```

## Create a Document with Mongoose

In `app.js`

Let's make ourselves an object to insert into our database. When we connect with an express app, our data will be coming in as an object from the browser.

```js
const myFirstTweet = {
  title: "Deep Thoughts",
  body: "Friends, I have been navel-gazing",
  author: "Karolin",
};
```

```js
Tweet.create(myFirstTweet)
// if database transaction succeeds
.then((tweet) => {
  console.log(tweet)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

Let's run this with
`node app.js`

We should see:

![created via mongoose](https://i.imgur.com/I0EbPuu.png)

Timestamps, deleted, and likes had default values, a unique \_id has been generated

Every time we run `node app.js` it will run the code, and thus insert this object over and over again. Let's not do that. Let's comment it out.

Let's insert many more tweets

```js
const manyTweets = [
  {
    title: "Deep Thoughts",
    body: "Friends, I have been navel-gazing",
    author: "Karolin",
  },
  {
    title: "Sage Advice",
    body: "Friends, I am vegan and so should you",
    author: "Karolin",
    likes: 20,
  },
  {
    title: "Whole Reality",
    body: "I shall deny friendship to anyone who does not exclusively shop at Whole Foods",
    author: "Karolin",
    likes: 40,
  },
  {
    title: "Organic",
    body: "Friends, I have spent $2300 to be one of the first people to own an organic smartphone",
    author: "Karolin",
    likes: 162,
  },
  {
    title: "Confusion",
    body: "Friends, why do you just respond with the word `dislike`? Surely you mean to click the like button?",
    author: "Karolin",
    likes: -100,
  },
  {
    title: "Vespa",
    body: "Friends, my Vespa has been upgraded to run on old french fry oil. Its top speed is now 11 mph",
    author: "Karolin",
    likes: 2,
  },
  {
    title: "Licensed",
    body: "Friends, I am now officially licensed to teach yogalates. Like this to get 10% off a private lesson",
    author: "Karolin",
    likes: 3,
  },
  {
    title: "Water",
    body: "Friends, I have been collecting rain water so I can indulge in locally sourced raw water. Ask me how",
    author: "Karolin",
  },
];
```

Let's insert all these tweets:

```js
Tweet.insertMany(manyTweets)
// if database transaction succeeds
.then((tweets) => {
  console.log(tweets)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})

```

- `node app.js`

and let's comment it out so we don't insert duplicates

## Find Documents with Mongoose

- Mongoose has 4 methods for this
- `find` - generic
- `findById` - finds by ID - great for Show routes!
- `findOne` - limits the search to the first document found
- [`where`](http://mongoosejs.com/docs/queries.html) - allows you to build queries, we won't cover this today

Let's find all

```js
Tweet.find({})
// if database transaction succeeds
.then((tweets) => {
  console.log(tweets)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

Let's limit the fields returned, the second argument allows us to pass a string with the fields we are interested in:

```js
Tweet.find({}, "title body")
// if database transaction succeeds
.then((tweets) => {
  console.log(tweets)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

Let's look for a specific tweet:

```js
Tweet.find({ title: "Water" })
// if database transaction succeeds
.then((tweet) => {
  console.log(tweet)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

We can also use advanced query options. Let's find the tweets that have 20 or more likes

```js
Tweet.find({ likes: { $gte: 20 } })
// if database transaction succeeds
.then((tweets) => {
  console.log(tweets)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

### Delete Documents with Mongoose

We have two copies of our first tweet and a few options to delete it

- `remove()` danger! Will remove all instances
- `findOneAndRemove()` - this seems like a great choice
- `.findByIdAndRemove()`- finds by ID - great for delete routes in an express app!

```js
Tweet.findOneAndRemove({ title: "Deep Thoughts" })
// if database transaction succeeds
.then((tweet) => {
  console.log(tweet)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

### Update Documents with Mongoose

Finally, we have a few options for updating

- `update()` - the most generic one
- `findOneAndUpdate()`- Let's us find one and update it
- `findByIdAndUpdate()` - Let's us find by ID and update - great for update/put routes in an express app!

If we want to have our updated document returned to us in the callback, we have to set an option of `{new: true}` as the third argument

```js
Tweet.findOneAndUpdate(
  { title: "Vespa" },
  { sponsored: true },
  { new: true })
// if database transaction succeeds
.then((tweet) => {
  console.log(tweet)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

We'll see the console.logged tweet will have the value of sponsored updated to true. Without `{new: true}` we would get the original unaltered tweet back.

### Intermediate

We can count how many tweets we have with likes greater than 20

```js
Tweet.countDocuments({ likes: { $gte: 20 } })
// if database transaction succeeds
.then((count) => {
  console.log(count)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

We can check out all the things we can do at the [Mongoose API docs](http://mongoosejs.com/docs/api.html)

### Advanced & New!

[It has an updated query builder that chains much like jQuery](http://mongoosejs.com/docs/queries.html).

Do a search, limit the number of returned queries to 2, sort them by title

```js
Tweet.find({ likes: { $gte: 20 } }, "title -_id")
  .limit(2)
  .sort("title")
  .exec()
// if database transaction succeeds
.then((tweets) => {
  console.log(tweets)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

<hr>

# Advanced Mongoose

## Lesson Objectives

1. Use helper functions to make your life easier
1. Extend the abilities of models
1. Extend the abilities of classes
1. Create a schema for properties of models that are objects
1. Reference other models by id
1. Create actions for models to execute during database actions
1. Use indexes to speed up searches

## Use helper functions to make your life easier

Mongoose's default find gives you an array of objects.  But what if you know you only want one object?  These convenience methods just give you one object without the usual array surrounding it.

```javascript
Article.findById('5757191bce5579b805705900')
// if database transaction succeeds
.then((article) => {
  console.log(article)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```
```javascript
Article.findOne({ author : 'Matt' })
// if database transaction succeeds
.then((tweet) => {
  console.log(tweet)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```
```javascript
Article.findByIdAndUpdate(
	'5757191bce5579b805705900', // id of what to update
	{ $set: { author: 'Matthew' } }, // how to update it
	{ new : true })
// if database transaction succeeds
.then((article) => {
  console.log(article)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```
```javascript
Article.findOneAndUpdate(
	{ author: 'Matt' }, // search criteria of what to update
	{ $set: { author: 'Matthew' } }, // how to update it
	{ new : true }) // tells findOneAndUpdate to return modified article, not the original
// if database transaction succeeds
.then((article) => {
  console.log(article)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```
```javascript
Article.findByIdAndRemove('5757191bce5579b805705900')
// if database transaction succeeds
.then((article) => {
  console.log(article)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```
```javascript

Article.findOneAndRemove({ author : 'Matt' })
// if database transaction succeeds
.then((article) => {
  console.log(article)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

## Extend the abilities of models

You can give all models using a specific schema extra properties and methods when creating that schema

```javascript
//after initial schema declaration
articleSchema.methods.longTitle = ()=>{
	return this.author + ": " + this.title;
}

//instantiate a model and then call it
const article = new Article();
console.log(article.longTitle());
```

## Extend the abilities of classes

You can create static methods for model constructor functions in their schema

```javascript

//after initial schema declaration
articleSchema.statics.search = function(name){ //because we use this here, we'll need an old-fashioned function
	//In this situation, it's like .find(), but it searches both title and author
	return this.find({
		$or : [
			{ title: new RegExp(name, 'i') },
			{ author: new RegExp(name, 'i') }
		]
	});
}

//call the static method
Article.search('Some')
// if database transaction succeeds
.then((article) => {
  console.log(article)
})
// if database transaction fails
.catch((error) => {
  console.log(error)
})
// close db connection either way
.finally(() => {
 db.close()
})
```

## Reference other models by id

Sub docs are difficult when it comes to updating, because all duplicates must be updated.  But we can reference by ID to get around this.

### Single ObjectId reference

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
mongoose.connect('mongodb://localhost:27017/test');

const articleSchema = new Schema({
	title: { type: String },
	author: { type: Schema.Types.ObjectId, ref: 'Author' } //the author property is just an id of another object
});

const authorSchema = new Schema({
	name: { type: String }
});

const Article = mongoose.model('Article', articleSchema);
const Author = mongoose.model('Author', authorSchema);

const matt = new Author({name: 'Matt'});
matt.save(()=>{
	const article1 = new Article({title:'Awesome Title', author: matt._id});
	article1.save(()=>{
		showAll();
	});
});

const showAll = ()=>{
	Article.find().populate('author').exec((error, article)=>{ //dynamically switch out any ids with the objects they reference
		console.log(article);
		mongoose.connection.close();
	})
};
```

### Arrays of ObjectId references

We can do the same with arrays of ids

```javascript
const authorSchema = new Schema({
	name: { type: String },
	articles: [{type: Schema.Types.ObjectId, ref: 'Articles'}]
});

const Article = mongoose.model('Article', articleSchema);
const Author = mongoose.model('Author', authorSchema);

const matt = new Author({name: 'Matt'});
let article_id;
matt.save(()=>{
	let article1 = new Article({title:'Awesome Title', author: matt._id});
	article1.save(()=>{
		article_id = article1._id;
		matt.articles.push(article1);
		matt.save(showAll);
	});
});

var showAll = (err, author)=>{
	Author.find().populate('articles').exec((err, authors)=>{ //dynamically switch out any ids with the objects they reference
		console.log(authors);
		mongoose.connection.close();
	});
};
```



## Create actions for models to execute during database actions

We can perform actions before and after database work

```javascript
articleSchema.pre('save', function(next){ //because we use this here, we'll need an old-fashioned function
	console.log(this);
	console.log('saving to backup database');
	next();
});
```

```
post
```javascript
articleSchema.post('save', function(next){ //because we use this here, we'll need an old-fashioned function
	console.log('saving complete');
});
```
