# Lesson 2: A help ticket server

Let's build a help ticket server and a Vue front end for submitting tickets. We'll assume a ticket has a name and a problem being reported.

You can run the back end code by:

```
cd lesson2/back-end
npm install
node server.js
```

You can run the front end code by:

```
cd lesson2/front-end
npm install
npm run serve
```

## Back end

The first step is to initialize a new project. You do
this with `npm init`.

```
mkdir lesson2
cd lesson2
mkdir back-end
cd back-end
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

The `front-end` directory contains the front end code, using Vue CLI.

This code is identical to the ticket server built in [Learning the Vue
CLI](https://github.com/BYU-CS-260/learning-vue-cli), with modifications to make
it work with a back end instead of storing the tickets in a global data
structure.

Let's run through the changes.

### No global data structure

First, if you look at `src/main.js`, there is no global data structure:

```
new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

### Adding and deleting tickets

Second, the home page in `src/views/Home.vue` now uses the
[axios](https://github.com/axios/axios) library to fetch data from the back end
instead of from the global data structure:

```
<script>
import axios from 'axios';
export default {
  name: 'home',
  data() {
    return {
      tickets: [],
    }
  },
  created() {
    this.getTickets();
  },
  methods: {
    async getTickets() {
      try {
        let response = await axios.get("/api/tickets");
        this.tickets = response.data;
        return true;
      } catch (error) {
        console.log(error);
      }
    },
    async deleteTicket(ticket) {
      try {
        await axios.delete("/api/tickets/" + ticket.id);
        this.getTickets();
        return true;
      } catch (error) {
        console.log(error);
      }
    }
  }
}
</script>
```

Notice that we store `tickets` as a property on the data object for the view. To
load the current set of tickets when the page is loaded, we use a `created()`
function. Think of this as initializing Vue for this page. This method calls
`getTickets()`, which is defined in the `methods` section.

To get tickets from the server, we could use `fetch`, but I like to use the
Axios library. To use it, I ran `npm install axios`, and then put an import
statement at the top of the `script` section of the view:

```
import axios from 'axios';
```

Then we need to make an async `getTickets()` function:

```
  async getTickets() {
      try {
        let response = await axios.get("/api/tickets");
        this.tickets = response.data;
        return true;
      } catch (error) {
        console.log(error);
      }
    },
```

This calls `axios.get` to get the data from our server's REST API and then store
the returned tickets in `this.tickets`. It uses `await` to wait for this to
happen instead of the older Promise `.then` syntax.

I also added a button to delete tickets in the `template` section of this component:

```
<button @click="deleteTicket(ticket)" />
```

And then the corresponding `deleteTicket()` method:

```
  async deleteTicket(ticket) {
    try {
      await axios.delete("/api/tickets/" + ticket.id);
      this.getTickets();
      return true;
    } catch (error) {
      console.log(error);
    }
  }
```

This method uses the REST API to delete a ticket. We supply the specific ID of
the ticket we are deleting by appending it to the URL. Notice that we get this
ticket ID from `index.html`. When we iterate through the list of tickets using
`v-for`, we can pass `ticket` to the `deleteTicket` method.

### Creating tickets

To create tickets, we modify the `addTicket()` function in `src/views/Create.view`:

```
    async addTicket() {
      try {
        await axios.post("/api/tickets", {
          name: this.name,
          problem: this.problem
        });
        this.name = "";
        this.problem = "";
        this.creating = false;
        return true;
      } catch (error) {
        console.log(error);
      }
```

This method uses the REST API to create a new ticket. Typically a REST API uses POST to create a new resource and UPDATE to edit it. Because this is a POST request, we supply a body that includes two properties needed by the API: `name`, and `problem`.

### Super Important -- Avoiding CORS errors

We're going to be running a server for the front end at `localhost:8080` and a
server for the back end at `localhost:3000`. This will cause CORS errors! CORS
is designed to make sure that if your browser is currently visiting
`amazon.com`, it can't also make a request to `stealmycreditcard.com`, a site
controlled by an attacker.

In our case, we control both of these websites, and need to ensure that our
front end server can talk to the back end server. We do this by creating a file
in `front-end/vue.config.js`. This configures the server that the Vue CLI starts
to `proxy` requests for the back end. This will tell it to send any request it
doesn't handle  to the back end server.

The `front-end/vue.config.js` file should contain:

```
module.exports = {
  devServer: {
    proxy: 'http://localhost:3000',
  }
}
```

You will  need to restart the Vue CLI front-end server for this to work. So if
you need to, use `control-c` to stop the Vue CLI server and then start it again
with

```
npm run serve
```

### Super Important -- Digital Ocean

When you setup a Vue CLI app to run on Digital Ocean, you will need to run a similar
setup. In this case, nginx will be the front end server (instead of npm run serve)
and you will still run `node ticket.js` as your back end server on port 3000. We
will show you later how to set this up.
