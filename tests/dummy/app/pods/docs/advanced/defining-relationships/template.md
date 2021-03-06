# Defining relationships

In modern versions of Mirage (0.3.3 and later), Mirage's relationships are inferred from your Ember Data schema.

If you are not using Ember Data or are on an older version of Mirage, you can manually define Mirage's Models so that you can take advantage of its many features that rely on the ORM, like Serializers and Factories.

To define associations, use the `belongsTo` and `hasMany` helpers. Each helper adds some dynamic methods to your model.

*belongsTo*

```js
// mirage/models/blog-post.js
import { Model, belongsTo } from 'ember-cli-mirage';

export default Model.extend({
  author: belongsTo()
});
```

This adds an `authorId` property to your `blogPost` model, as well as some methods for working with the associated `author` model:

```js
blogPost.authorId;                // 1
blogPost.authorId = 2;            // updates the relationship
blogPost.author;                  // Author instance
blogPost.author = anotherAuthor;
blogPost.newAuthor(attrs);        // new unsaved author
blogPost.createAuthor(attrs);     // new saved author (updates blogPost.authorId in memory only)
```
Note that when a child calls `child.createParent`, the new parent is immediately saved to the `db`, but the child's foreign key is updated *on this instance only*, and is not immediately persisted to the database.

In other words, `blogPost.createAuthor` will create a new `author` record, insert it into the `db`, and update the `blogPost.authorId` in memory, but if you were to fetch the `blogPost` from the `db` again, the relationship would not be persisted.

To persist the new foreign key, you would call `blogPost.save()` after creating the new author.

*hasMany*

```js
// mirage/models/blog-post.js
import { Model, hasMany } from 'ember-cli-mirage';

export default Model.extend({
  comments: hasMany()
});
```

This adds a `commentIds` property to the `blogPost` model, as well as some methods for working with the associated `comments` collection:

```js
blogPost.commentIds;                      // [1, 2, 3]
blogPost.commentIds = [2, 3];             // updates the relationship
blogPost.comments;                        // array of related comments
blogPost.comments = [comment1, comment2]; // updates the relationship
blogPost.newComment(attrs);               // new unsaved comment
blogPost.createComment(attrs);            // new saved comment (comment.blogPostId is set)
```

### Association options

*modelName*

If your associations model has a different name than the association itself, you can specify the `modelName` on the association.

For example,

```js
// mirage/models/blog-post.js
import { Model, belongsTo, hasMany } from 'ember-cli-mirage';

export default Model.extend({
  author: belongsTo('user'),
  comments: hasMany('annotation')
});
```

would add all the named `author` and `comment` methods as listed above, but use `user` and `annotation` models for the actual relationships.

*inverse*

Sometimes a `hasMany` relationship can point to a model with multiple associations of the same type. You can use the `inverse` option to specify which property on the related model is the inverse of the given relationship:

```js
// app/models/talk.js
export default Model.extend({
  primaryEvent: belongsTo('event'),
  secondaryEvent: belongsTo('event'),
});

// app/models/event.js
export default Model.extend({
  talks: hasMany('talk', { inverse: 'primaryEvent' }),
});
```

*polymorphic*

You can specify whether an association is a polymorphic association by passing `{ polymorphic: true }` as an option.

For example, say you have a `Comment` that can belong to a `BlogPost` or a `Picture`. Here's how the model definitions would look:

```js
// app/models/comment.js
export default Model.extend({
  commentable: belongsTo({ polymorphic: true })
});

// app/models/blog-post.js
export default Model.extend({
  comments: hasMany()
});

// app/models/picture.js
export default Model.extend({
  comments: hasMany()
});
```

Note that `commentable` doesn't need a type (there's no validation done on which types of models can exist on that association).

Polymorphic associations have slightly different method signatures for their foreign keys and build/create methods.

```js
let comment = schema.comments.create({ text: "foo" });

comment.buildCommentable('post', { title: 'Lorem Ipsum' });
comment.createCommentable('post', { title: 'Lorem Ipsum' });

// getter
comment.commentableId; // { id: 1, type: 'blog-post' }

// setter
comment.commentableId = { id: 2, type: 'picture' };
```

Has-many asssociations can also be polymorphic:

```js
// app/models/user.js
export default Model.extend({
  things: hasMany({ polymorphic: true })
});

// app/models/car.js
export default Model.extend({
});

// app/models/watch.js
export default Model.extend({
});

let user = schema.users.create({ name: "Sam" });

user.buildThing('car', { attrs });
user.createThing('watch', { attrs });

// getter
user.thingIds; // [ { id: 1, type: 'car' }, { id: 3, type: 'watch' }, ... ]

// setter
user.thingIds = [ { id: 2, type: 'watch' }, ... ];
```
