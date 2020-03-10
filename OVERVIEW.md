# Overview

Node.js is a Javascript *runtime*. This means that it contains
everything you need to run a JavaScript program, just like your
browser.  You can use Node.js to execute a JavaScript program on your
laptop or on a server.

The core of Node.js translates your JavaScript program into machine
language so that it can run faster on your machine, similar to a
compiler. Node.js also includes event processing, known as an event
loop, so that it can efficiently handle many events. Any time
JavaScript creates an event (e.g. with a Promise), then Node.js will
keep track of that event and execute the callback function (e.g. in
*then*) when the event is finished.

A good illustration of this is the following code:

```
const fs = require('fs');
const data = fs.readFileSync('/file.md'); // blocks here until file is read
```

This is a *synchronous* or *blocking* read from a file. JavaScript
will block until the entire file is read and stored in the constant
called `data`.

We can do this asynchronously instead:

```
const fs = require('fs');
fs.readFile('/file.md', (error, data) => {
  if (error) throw error;
  // do something with the data
});
// more work here
```

This is an *asynchronous* or *non-blocking* read from a
file. JavaScript will create an event to read the file, and when that
event is finished, it will execute the anonymous function (it takes
error and data as parameters). Instead of waiting for this event to
finish, JavaScript will execute the part that reads *more work here*.

See our prior activity [JavaScript Promises](https://github.com/BYU-CS-260/promises)
for a refresher on this concept.

## Web server

Node has a built-in HTTP library that you can use to create a web server.
For example:

```
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

This creates a simple (and not very useful) web server that always
returns a plain text file that contains the string "Hello World". It
always returns a 200 HTTP status code.

## Package Manager

Node.js includes a large number of packages that you can install with
a package manager. The standard package manager is called NPM (Node
Package Manager) and you can use it with the `npm` command. For example,

```
npm install axios
```

This will install the axios library for making Ajax requests. Packages
are installed in a directory called `node_modules`. The names and
versions of packages that you install are saved in a file called
`package.json`. If you clone a repository that uses Node.js, you can
install all of its packages (from `package.json`) using:

```
npm install
```

We recommend that you do *not* include `node_modules` in your git
repositories. This would force everyone who clones your repository to
also clone all the packages you use. Put the following into a file
called `.gitignore`, which you can add to your repository:

```
node_modules
```

You can also use other package managers instead of NPM, such as
[yarn](https://yarnpkg.com/en/).

## Express

Express is a library for Node.js that makes it easy to build a web application.
We will use it primarily for providing an API for your server.

For example, if you build a web site for an art museum, your server
may need an API that does some of the following:

Route | HTTP method | Description
------|-------------|-------------
/api/art | GET | get all the art pieces held by the museum
/api/art | POST | create a new art piece held by the museum
/api/art/:id | GET | get a single art piece
/api/art/:id | PUT | update the information about a single art piece
/api/art/:id | DELETE | delete the information about a single art piece

You can use Express to implement this API, so that your front end can
make calls to the server:

```
let art = await axios.get('/api/art/5');
```
