# Learning Node.js and Express

You can run this code by:

```
node server.js
```

## Lesson 2: A help ticket server

Let's build a help ticket server and a Vue front end for submitting tickets. We'll assume a ticket has a name and a problem being reported.

The first step is to initialize a new project. You do
this with `npm init`.

```
mkdir lesson2
cd lesson2
npm init
```

Like the first lesson, we'll answer these questions:

```
package name: (lesson2)
version: (1.0.0)
description: help ticket server
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
```

Now we need to install Express and an npm library for parsing the body of POST requests.

```
npm install express body-parser
```

Next, create a file called `tickets.js` with the following code:

```
const express = require('express');
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
```

This includes the modules we're using and initializes them.

```
app.use(express.static('public'));
```

This tells Express that it should serve any files in the `public` directory as if they were just
static files on a web server. We'll put our Vue code here later on.

```
let tickets = [];
let id = 0;
```

Since we don't have a database setup yet, we're just going to store our tickets in a global variable.

```
app.get('/api/tickets', (req, res) => {
  res.send(tickets);
});
```

This is the REST endpoint for getting all the tickets in the system. We just send our list,
which by default comes with a 200 OK response.

```
app.post('/api/tickets', (req, res) => {
  id = id + 1;
  let ticket = {
    id: id,
    name: req.body.name,
    problem: req.body.problem
  };
  tickets.push(ticket);
  res.send(ticket);
});
```

This is the REST endpoint for creating a new ticket. We get the parameters from the request body,
create a new ticket, then send back the same ticket we created in a 200 OK response. We've left out
some error checking---we should check whether the request body includes the desired information.

```
app.delete('/api/tickets/:id', (req, res) => {
  let id = parseInt(req.params.id);
  let removeIndex = tickets.map(ticket => {
      return ticket.id;
    })
    .indexOf(id);
  if (removeIndex === -1) {
    res.status(404)
      .send("Sorry, that ticket doesn't exist");
    return;
  }
  tickets.splice(removeIndex, 1);
  res.sendStatus(200);
});
```

This is the REST endpoint for deleting a ticket. The ID is passed in the URL so we use a different
method to parse it. We check whether this ID is present and return a 404 error if it doesn't.
Otherwise, we remove it and return 200 OK.

```
app.listen(3000, () => console.log('Server listening on port 3000!'));
```

This starts the server on port 3000.

## Testing with curl

Run the server:

```
node tickets.js
```

Then you can test it with curl:

```
$ curl -d '{"name":"Daniel","problem":"Nothing works! This software is junk!"}' -H "Content-Type: application/json" -X POST localhost:3000/api/tickets
{"id":1,"name":"Daniel","problem":"Nothing works! This software is junk!"}

$ curl localhost:3000/api/tickets
[{"id":1,"name":"Daniel","problem":"Nothing works! This software is junk!"}]

$ curl -d '{"name":"Daniel","problem":"Never mind, this system is cool. It was a feature, not a bug!"}' -H "Content-Type: application/json" -X POST localhost:3000/api/tickets
{"id":2,"name":"Daniel","problem":"Never mind, this system is cool. It was a feature, not a bug!"}

$ curl localhost:3000/api/tickets
[{"id":1,"name":"Daniel","problem":"Nothing works! This software is junk!"},{"id":2,"name":"Daniel","problem":"Never mind, this system is cool. It was a feature, not a bug!"}]
```


## Vue front end

In the `public` directory, you'll find a Vue front end for adding and deleting tickets.  This code
should be easy to follow given what we've done with Vue so far.

When you run `node tickets.js`, then the node server will deliver
`/public/index.html` because we have included this line:

```
app.use(express.static('public'));
```

This causes the browser to load the HTML, CSS, and JavaScript for the front end. Inside this code, you'll see some methods that use `axios` to call the REST API defined by the back end in `tickets.js`.

Let's run through each of these methods:

```
    async getTickets() {
      try {
        let response = await axios.get("http://localhost:3000/api/tickets");
        this.tickets = response.data;
        return true;
      } catch (error) {
        console.log(error);
      }
    },
```

This method uses the REST API to get the current list of tickets. It puts the response data directly into the `tickets` property, which is displayed in `index.html`.

Note that we call this method when the Vue instance is created. See the `created()` section. We also call it whenever we change the set of tickets.

```
    async addTicket() {
      try {
        let response = await axios.post("http://localhost:3000/api/tickets", {
          name: this.addedName,
          problem: this.addedProblem
        });
        this.addedName = "";
        this.addedProblem = "";
        this.getTickets();
        return true;
      } catch (error) {
        console.log(error);
      }
    },
```

This method uses the REST API to create a new ticket. Typically a REST API uses POST to create a new resource and UPDATE to edit it. Because this is a POST request, we supply a body that includes two properties needed by the API: `name`, and `problem`.

Once we create the new ticket, we call the API to fetch the list of tickets.

```
    async deleteTicket(ticket) {
      try {
        let response = axios.delete("http://localhost:3000/api/tickets/" + ticket.id);
        this.getTickets();
        return true;
      } catch (error) {
        console.log(error);
      }
    }

```

This method uses the REST API to delete a ticket. We supply the specific ID of the ticket we are deleting by appending it to the URL. Notice that we get this ticket ID from `index.html`. When we iterate through the list of tickets using `v-for`, we can pass `ticket` to the `deleteTicket` method.
