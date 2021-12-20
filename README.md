@json-schema-org/tests
======================
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg)](https://github.com/json-schema-org/.github/blob/main/CODE_OF_CONDUCT.md)
[![Project Status: Inactive – The project has reached a stable, usable state but is no longer being actively developed; support/maintenance will be provided as time allows.](https://www.repostatus.org/badges/latest/inactive.svg)](https://www.repostatus.org/#inactive)
[![Financial Contributors on Open Collective](https://opencollective.com/json-schema/all/badge.svg?label=financial+contributors)](https://opencollective.com/json-schema) 

NPM / node.js-specific support for the [JSON Schema test suite](https://github.com/json-schema-org/JSON-Schema-Test-Suite)

[![Build Status](https://travis-ci.org/json-schema-org/json-schema-test-suite-npm.svg?branch=master)](https://travis-ci.org/json-schema-org/json-schema-test-suite-npm)

The JSON Schema Test Suite is meant to be a language agnostic test suite for testing JSON Schema validation libraries.
It is generally added to projects as a git submodule. However, to simplify things for Node.js developers, the test suite has also
been made available as an npm package.

```sh
npm install @json-schema-org/tests
```

### Usage:

There are a number of ways to load tests from the suite:

```js
const testSuite = require('@json-schema-org/tests');

// this will load all (required and optional) draft6 tests
const tests = testSuite.loadSync();

// optional `filter` is a function that takes 3 arguments (filename, parent, optional)
// and returns true if the test should be included. The optional argument is true
// for all files under the `<draft>/optional` directory.
// optional `draft` should be either `'draft3'`, `'draft4'` or `'draft6'`

const tests = testSuite.loadSync(filter, draft);

// convenience functions:

// The following take an optional `filter` as described previously (undefined will load all tests)
const draft3 = testSuite.draft3();
const draft4 = testSuite.draft4();
const draft6 = testSuite.draft6();

// The following take an optional `draft` argument (defaults to 'draft6')
const all = testSuite.loadAllSync();
const required = testSuite.loadRequiredSync();
const optional = testSuite.loadOptionalSync();
```


The return value of these functions is an array of objects that correspond to each file under `tests/<draft>` that
passed the filter (the default is all, so the array will also include all the optional files).

Each object has the following structure (using `tests/draft4/additionalItems.json` as an example):

```js
{
  name:    'additionalItems',
  file:     'additionalItems.json',
  optional: false,  // true if a file under the optional directory
  schemas:  []
}
```

The `schemas` property contains the array of objects loaded from the test file.
Each object consists of a schema and description, along with a number of tests used for validation. Using the first schema object in the array from `tests/draft4/additionalItems.json` as an example:

```js
{
  description: 'additionalItems as schema',
  schema: {
    items: [{}],
    additionalItems: { type: "integer" }
  },
  tests: [
    {
      description: "additional items match schema",
      data: [ null, 2, 3, 4 ],
      valid: true
    },
    {
      description: "additional items do not match schema",
      data: [ null, 2, 3, "foo" ],
      valid: false
    }
  ]
}
```

### Testing a JSON Validator

You can apply a validator against all the tests. You need to create a validator factory function that takes a JSON schema and an options argument, and returns an object with a validate method. The validate function should take a JSON object to be validated against the schema. It should return an object with a valid property set to true or false, and if not valid, an errors property that is an array of one or more validation errors.

The following are examples of `Tiny Validator (tv4)` and `z-schema` validator factories used by the unit test.


#### tv4

```js
const tv4 = require('tv4');

const tv4Factory = function (schema, options) {
  return {
    validate: function (json) {
      try {
        const valid = tv4.validate(json, schema);
        return valid ? { valid: true } : { valid: false, errors: [ tv4.error ] };
      } catch (err) {
        return { valid: false, errors: [err.message] };
      }
    }
  };
};
```

#### ZSchema

```js
const ZSchema = require('z-schema');

const zschemaFactory = function (schema, options) {
  const zschema = new ZSchema(options);

  return {
    validate: function (json) {
      try {
        const valid = zschema.validate(json, schema);
        return valid ? { valid: true } : { valid: false, errors: zschema.getLastErrors() };
      } catch (err) {
        return { valid: false, errors: [err.message] };
      }
    }
  };
};
```

#### Testing the Validator

Using a validator factory as described above, you can test it as follows.

```js
const testSuite = require('json-schema-test-suite');

const tests = testSuite.testSync(factory);
```

The `tests` return value is as described previously in the Usage section, with an additional property for each test object that corresponds to the test result:

```js
{
  description: 'additionalItems as schema',
  schema: {
    items: [{}],
    additionalItems: { type: "integer" }
  },
  tests: [
    {
      description: "additional items match schema",
      data: [ null, 2, 3, 4 ],
      valid: true,
      result: {
        valid: false,
        errors: [ ... ]
      }
    },
    {
      description: "additional items do not match schema",
      data: [ null, 2, 3, "foo" ],
      valid: false,
      result: {
        true
      }
    }
  ]
}
```

### Tests

You can run the tests by doing:

```sh
npm test
```
