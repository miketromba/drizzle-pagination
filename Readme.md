# drizzle-pagination
Easily add cursor pagination to your drizzle-orm queries.

## Installation
```bash
npm install drizzle-pagination
```

## Usage

### With a single, unique cursor
Use a single cursor when the column that you want to sort your results by is guaranteed to be unique (for example, an `id` column).
```js
import { eq } from 'drizzle-orm'
import { withCursorPagination } from 'drizzle-pagination'

const page = await db.query.post.findMany(
    withCursorPagination({
        where: eq(schema.post.status, 'published'),
        limit: 32,
        cursors: [
            [
                schema.post.id, // Column to use for cursor 
                'asc', // Sort order ('asc' or 'desc')
                '94b5a795-5af4-40c3-8db8-a1c33906f5af' // Cursor value
            ]
        ]
    })
)
```
> **Warning** - 
> When using a single cursor, make sure it is a unique column - otherwise you may get unexpected results.

### With two cursors (one non-unique, sequential cursor, and a unique cursor as a fallback)
Use two cursors when you want to order your results by a column that is not unique. For example:
```js
const page = await db.query.post.findMany(
    withCursorPagination({
        where: eq(schema.post.status, 'published'),
        limit: 32,
        cursors: [
            // Non-unique, sequential column
            [schema.post.publishedAt, 'desc', '2023-07-14 16:00:00'],
            // Unique column as a fallback so you get a stable sort order
            [schema.post.id, 'asc', '94b5a795-5af4-40c3-8db8-a1c33906f5af']
        ]
    })
)
```
> **Warning** - 
> When using two cursors, your second cursor should always be a unique column to ensure a stable sort order.

### Using spread operator syntax
Use spread syntax to combine the `withCursorPagination` helper with other query inputs, like the `with` option, for example. 
```js
const page = await db.query.post.findMany({
    ...withCursorPagination({
        where: eq(schema.post.status, 'published'),
        limit: 32,
        cursors: [
            [schema.post.publishedAt, 'desc', '2023-07-14 16:00:00'],
            [schema.post.id, 'asc', '94b5a795-5af4-40c3-8db8-a1c33906f5af']
        ]
    }),
    with: { author: true }
})
```

### What about nullable columns?
Paginating with a nullable column may have varied results depending on the database/driver you're using. If possible, try to use non-nullable columns for your pagination. For example:

#### Instead of a nullable `string` column:
Set the default value to an empty string. All columns with the empty string will appear at the top of an ascending sort. 

#### Instead of a nullable `datetime` column:

Set the default value to "1970-01-01 00:00:00" (the oldest possible value in most databases). Then, in your application, instead of doing a null check, simply check for the default value.

```js
const date = someDate === "1970-01-01 00:00:00" ? null : someDate
```

This will ensure that the order of your results will always be what you expect.

### Can I use more than two cursors?
**Not yet.** Right now this library only supports using one or two cursors. This should cover the majority of use cases. That being said, a PR adding support for a dynamic amount of cursors is welcome! E.g. A common use-case for 3+ cursors might be: `[firstName, lastName, id]`.

## Contributing
PRs are welcome!

## License
This package is distributed under the [MIT license](https://opensource.org/license/mit/).