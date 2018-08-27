### Summary (TL;DR)

* Import NPM dependency `slugify`
* Use it, e.g. `slugify(someString).toLowerCase()`

### Details:

Create a new file named `./models.js` and copy the following code inside of it.

```typescript
const Sequelize = require("sequelize");

const db = new Sequelize({
  dialect: "sqlite",
  storage: "./database.sqlite"
});

const Post = db.define("post", {
  title: { type: Sequelize.STRING },
  body: { type: Sequelize.TEXT },
  authorId: { type: Sequelize.STRING },
  slug: { type: Sequelize.STRING }  // <--- THIS
});

db.sync();

module.exports = { Post };
```

This code initializes a new SQLite database that will be used to store the blog data and also defines a model called `Post` which stores blog posts in the database. Each post has a title, a body, an author ID, and a slug field.

- The `title` field will hold the title of a post, eg: "A Great Article"
- The `body` field will hold the body of the article as HTML, eg: "\<p\>My first post!\</p\>"
- The `authorId` field will store the author’s unique ID. This is a common pattern in relational databases: store just the identifier of a linked resource so you can look up the author’s most up-to-date information later.
- The `slug` field will store the URL-friendly version of the post’s title, eg: "a-great-article"

**NOTE**: If you’ve never used SQLite before, it’s amazing. It’s a database that stores your data in a single file. It’s great for building applications that don’t require a large amount of concurrency, like this simple blog.

The call to `db.sync();` at the bottom of the file will automatically create the database and all of the necessary tables once this JavaScript code runs.

Somewhere else in your code, e.g. `./blog-controller.js`

```typescript
const slugify = require("slugify");
const models = require("./models");

const title = 'A Great Article!';
const body = 'Some HTML Content...';

// Create a new post
let newPost = await models.Post.create({
  title,
  body,
  slug: slugify(title).toLowerCase()  // 'A Great Article!' ---> 'a-great-article'
});

// You can now use newPost.slug in a URL, eg: /api/posts/a-great-article
```

