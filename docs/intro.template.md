!OUTPUT ../readme.md

[<img src="https://github.com/brillout/wildcard-api/raw/master/docs/images/logo.svg?sanitize=true" align="left" height="148" width="181">](https://github.com/brillout/wildcard-api)

# Wildcard API

*APIs. Cheap 'n Easy.*

<br/>
<br/>

Let your client load data from your server in an easy, flexible, and performant way.

It's super easy:

~~~js
// Node.js Server
const {endpoints} = require('wildcard-api');
const db = require('./db');

// We define a `getTodos` function on the server
endpoints.getTodos = async function() {
  const todos = await db.query("SELECT text FROM todos;");
  return todos;
};
~~~
~~~js
// Browser
import {endpoints} from 'wildcard-api/client';

(async () => {
  // Wildcard makes our `getTodos` function
  // available in the browser
  const todos = await endpoints.getTodos();

  document.body.textContent =
    todos
    .map(todo => ' - '+todo.text)
    .join('\n');
})();
~~~

You simply define functions on the server and Wildcard makes them availabe in the browser.
<br/>
(Behind the curtain, Wildcard makes an HTTP request and uses JSON.)

Creating a new API endpoint is as easy as creating a new function.

Wildcard is ideal for rapid prototyping, quickly delivering an MVP, and fast development iterations.

#### Contents

 - [Example](#example)
 - [Tailored Approach](#tailored-approach)
 - [Wildcard vs REST vs GraphQL](#wildcard-vs-rest-vs-graphql)
 - [Getting Started](#getting-started)


## Example

Let's consider an API for a simple todo app.

~~~js
!INLINE ../example/api/view-endpoints --hide-source-path
~~~

(This is a snippet of the example at [./example](/example/).)

Our endpoints are tailored to our frontend:
The endpoint `getCompletedTodosPageData` returns exactly and only the data needed
by the page `completedTodosPage` that shows all completed todos, and
the endpoint `getLandingPageData` returns exactly and only the data needed
by the landing page that shows user information and all todos that aren't completed.

We could have created generic endpoints instead:

~~~js
!INLINE ../example/api/generic-endpoints --hide-source-path
~~~

But we deliberately choose to implement a tailored API instead of a generic API.

Let's see why.

## Tailored Approach

To see why a tailored API makes sense,
let's imagine we want to implement a new feature for our todo app:
We want a page that shows all the todos that the user has shared with someone.

Getting our list of shared todos is very difficult
with a RESTful/GraphQL API.
(You'd need to extend the RESTful/GraphQL API in a clunky and unnatural way.)
In general,
any data requirement that doesn't fit the schema of a generic API is problematic.

But with a tailored API, it's easy:
We simply create a new endpoint that uses SQL to query the list of shared todos.
We can "directly" run SQL queries and we don't have to go over the indirection of a generic API.

###### Full backend power

A frontend developer can use any arbitrary SQL query to retrieve whatever the frontend needs.
SQL is much (much!) more powerful than any RESTful or GraphQL API.
Behind the curtain a RESTful/GraphQL API will perform SQL queries anyways.
Going over a generic API is simply an indirection and a net loss in power.

One way to think about Wildcard is that it directly exposes the whole power of your backend to the client in a secure way.
Not only SQL queries,
but also NoSQL queries,
cross-origin HTTP requests,
etc.

Wildcard is also efficient:
A tailored endpoint can return exactly and only the data the client needs.

###### But...

A potential downside of a tailored API
is if you have many clients with many distinct data requirements:
Maintaining a huge amount of tailored endpoints can become cumbersome.

In our todo app example above,
where the browser is our only client and where we have only few endpoints,
there are virtually no reasons to not prefer a tailored API over a generic one.

On the other side of the spectrum,
if you want third parties to access your data,
then you basically have an unlimited number of clients
and a generic API is required.

## Wildcard vs REST vs GraphQL

|                           | Wildcard API \*  | RESTful API \*\* | GraphQL API |
| ------------------------- | :--------------: | :--------------: | :---------: |
| Easy to setup             | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/minus.svg?sanitize=true'/> | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/minus.svg?sanitize=true'/> <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/minus.svg?sanitize=true'/> |
| Performant                | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/minus.svg?sanitize=true'/> | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> |
| Flexible (few endpoints)  | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/minus.svg?sanitize=true'/><img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/minus.svg?sanitize=true'/> | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/minus.svg?sanitize=true'/> |
| Flexible (many endpoints) | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/minus.svg?sanitize=true'/> | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> | <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> <img src='https://github.com/brillout/wildcard-api/raw/master/docs/images/plus.svg?sanitize=true'/> |

\* Following the [Tailored Approach](#tailored-approach)
<br/>
\*\* Following at least [REST level-1](https://martinfowler.com/articles/richardsonMaturityModel.html#level1)

With many endpoints we denote a high number of endpoints
to the point of being unmanageable.
The criteria is this:
For a database change, how much effort is required to adapt the affected endpoints?
With a large amount of endpoints,
that effort can become high and using REST/GraphQL can be more appropriate.

Rough estimate of when to use what:
- A **prototype** typically has few endpoints and
  **Wildcard** is certainly the better choice.
  Example: You are a startup and you need to ship an MVP ASAP.
- A **medium-sized application** typically has a manageable amount of endpoints and
  **Wildcard** is most likely the better choice.
  Example: A team of 4-5 developers implementing a Q&A website like StackOverflow.
- A **large application** may have so many endpoints that maintaining a Wildcard API can become cumbersome and
  **REST/GraphQL** can make more sense.

You can implement your prototype with Wildcard
and later migrate to REST/GraphQL
if your application grows into having too many endpoints.
Migration is easily manageable by progressively replacing Wildcard endpoints with RESTful/GraphQL endpoints.

Also, combining a Wildcard API with a RESTful/GraphQL API can be a fruitful strategy.
For example, a RESTful API for third-party clients combined with a Wildcard API for your clients.
Or a GraphQL API for most of your data requirements combined with a Wildcard API
for couple of data requirements that cannot be fulfilled with your GraphQL API.


## Getting Started

Work-in-progress.

(Although you can go ahead and use it. `npm install wildcard-api` then define functions on `const {endpoints} = require('wildcard-api')` on the server, then use them in the browser with `import {endpoints} from 'wildcard-api/client';`. See the [example](/example/) to see how to integrate with Express (or any other server framework).)
