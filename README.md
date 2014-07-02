# Elasticsearch streams

Expose a Writeable stream for bulk commands and a Readable stream from
hits and documents responses.

Use case: pipe to and from levelup, pouchdb and other friends.

The client that executes the requests is more or less abstracted at the moment.
Examples and tests are using the official client.

# Examples:

## Stream random records into Elasticsearch
```
var WritableBulk = require('elasticsearch-streams').WritableBulk;
var client = new require('elasticsearch').Client();

var bulkExec = function(bulkCmds, callback) {
  client.bulk({
    index : 'myindex',
    type  : 'mytype',
    body  : bulkCmds
  }, callback);
};
var ws = new WritableBulk(bulkExec);

// stream 42 random records into ES
require('random-document-stream')(42).pipe(ws);
```

## Stream search results from Elasticsearch
```
var ReadableSearch = require('elasticsearch-streams').ReadableSearch;
var client = new require('elasticsearch').Client();

var search = {
  index: 'myindex',
  from: 0,
  size: 12,
  body: {
    query: { match_all: {} }
  }
};
var queryExec = client.search.bind(client);

var rs = new ReadableSearch(queryExec, search);
var ws = new require('stream').Writable({objectMode:true});
ws._write = function(chunk, enc, next) {
  console.log('a hit', hit);
  next();
};

rs.pipe(ws);
```

## TODO
### Short term
* Document more
* Multi-get as a duplex
* Bulk document errors emitted as errors.
* Search stream: access to total, aggregations etc

## Later
Streaming http client or transport

# LICENSE
elasticsearch-streams is freely distributable under the terms of the MIT license.

Copyright (c) 2014 Sutoiku, Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
documentation files (the "Software"), to deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit
persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the
Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE
WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
