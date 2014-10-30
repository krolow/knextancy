# Knextancy - knex with multitenancy

Small library that provides a way of implementing multi-tenancy using table prefixes.

It has a very simple API:

```js
var knextancy = require('knextancy');

knextancy.tenant(knex, tenantId).then(function (tenantKnex) {

  tenantKnex('$tenant:users').where({
    first_name: 'Test',
    last_name:  'User'
  }).select('id')

});
```

Its `tenant` method expects a `knex` instance and a `tenantId` and returns **Promise** for a special `tenantKnex` instance that scopes every queries to the particular tenant.

The only requirement is that every query is written using the special `$tenant:` prefix for every table name.

## Migrations

**Knextancy** assures that all migrations are ran on a tenant's tables before returning its `knex` instance.

This special naming convention also applies while writing migrations, for example:

```js
'use strict';

exports.up = function(knex, Promise) {
  return knex.schema.createTable('$tenant:users', function (table) {
    table.string('name');
  });
};

exports.down = function(knex, Promise) {
  return knex.schema.dropTable('users');
};
```

## Connect Middleware

It also provides a handy [Connect](https://github.com/senchalabs/connect#readme) middleware that automatically creates a `knex` instance and attaches it to the `request` object for a kiven tetant based on a special HTTP header.

Bellow is a usage example:

```js
var app = express();

app.use(knextancy.middleware(knex, { header: 'x-client-id' }));

app.get('/', function (req, res, next) {
  req.knex.select().from('$tenant:users').then(function (users) {
    res.send(users);
  }, next);
});
```

The `knextancy.middleware` expects two parameters:

* `knex` instance;
* `options.header` the name of the HTTP header that will contain the tenant id.

## Troubleshooting

### Error: proxies not supported on this platform

Knextancy requires a special feature from the next release of JavaScript called [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy).

To enable it in Node.js (in its current version 0.10), it is required to enable the **harmony** flag while executing your appilation:

```js
node --harmony index.js
```