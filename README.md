# hapi-mongojs

[![NPM version][npm-image]][npm-url] [![Build Status][travis-image]][travis-url] [![codecov.io][codecov-image]][codecov-url]

A tiny plugin to share a common MongoDB connection pool across the whole Hapi server using [mongojs](https://github.com/mafintosh/mongojs).

You can ensure collection indexes.

### Setup
`npm install --save hapi-mongojs`

### Example
```javascript

const Hapi = require('hapi');
const Boom = require('boom');
// IMPORT NPM DEPENDENCY
const mongojs = require('hapi-mongojs');

// ADD PLUGINS CONFIG
const plugins = [
  {
    register: mongojs,
    options: {
      url: 'mongodb://localhost:27017/myDatabase',
      // ENSURE COLLECTION INDEXES (OPTIONAL)
      collections: [{
        name: 'myCollection1',
        indexes: [{
          keys: {
            'aField': 1
          },
          options: {
            'v': 1,
            'unique': true,
            'name': 'afield_idx',
            'ns': 'database.myCollection1'
          }
        }]
      }]
    }
  }
];

const server = new Hapi.Server();

server.connection({
  host: 'localhost',
  port: 8888
});

server.route([
  {
    method: 'GET',
    path: '/example',
    handler: function (request, reply) {

      // GET DB CONNECTION
      const myCollection = mongojs.db().collection('myCollection1');

      // EXECUTE QUERY
      myCollection.find((error, value) => {
        if (error) {
          return reply(Boom.badData('Internal MongoDB error', error));
        }
        reply(value);
      });

    }
  }
]);

server.register(plugins, (err) => {
  if (err) {
    console.error(err);
    throw err;
  }

  server.start((err) => {
    if (err) {
      console.error(err);
      throw err;
    }
    console.log('info', `Server running at: ${server.info.uri}`);
  });
});

```

### Start MongoDB and run the example
```bash
npm run example:install
npm run example:start

# verify if server is running
http://localhost:8888/status
# example
http://localhost:8888/example
```

[npm-image]: https://img.shields.io/npm/v/hapi-mongojs.svg
[npm-url]: https://www.npmjs.com/package/hapi-mongojs
[travis-image]: https://travis-ci.org/niqdev/hapi-mongojs.svg?branch=master
[travis-url]: https://travis-ci.org/niqdev/hapi-mongojs
[codecov-image]: https://codecov.io/github/niqdev/hapi-mongojs/coverage.svg?branch=master
[codecov-url]: https://codecov.io/github/niqdev/hapi-mongojs?branch=master
