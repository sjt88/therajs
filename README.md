[![npm version](https://badge.fury.io/js/thera.svg)](https://badge.fury.io/js/thera)
[![Build Status](https://travis-ci.org/sjt88/therajs.svg?branch=master)](https://travis-ci.org/sjt88/therajs)
![Dependencies](https://david-dm.org/sjt88/therajs.svg)
[![Coverage Status](https://coveralls.io/repos/github/sjt88/therajs/badge.svg?branch=develop)](https://coveralls.io/github/sjt88/therajs?branch=develop)

## DEPRECATED - The EVE Online XML API which this library consumed was deprecated a while ago in favour of [ESI](https://esi.evetech.net/ui/).

# Thera.js
#### An ES6 Promise driven EVE Online API client

### About
Thera is a Promise driven Eve Online API client for node.js written in ES6.  

Thera respects the Eve Online APIs Cache Timers and features automatic caching of retrieved resources. It includes and by default uses an in-memory cache, however this can easily be overridden with a custom cache provided it exposes the methods detailed in the .cache section under the API heading.

### Installation
```bash
npm install --save thera
```

___

### API Clients
Currently thera only contains an XML API client class - thera.XmlClient.  

In the future it will be extended to include a CREST API client component & any other EVE related API clients I might feel like creating. 

___

### Usage

#### Basic Usage

```javascript
import thera from 'thera';

var xmlClient = new thera.XmlClient({
  keyID: '123456',
  vCode: 'asdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasda'
})

xmlClient.get('account/Characters').then(response => {
  // do something with the response
})
```

#### Method/Promise Chaining
```javascript
var xmlClient = new thera.XmlClient();

xmlClient
  .setKeyID('5052100')
  .setVCode('jqPQnIj5FnT6obJulCdnQF514NFlIW2sjV3M6EFfWAxLG9tVNGXXbhgcdp9Rbzon')
  .get('account/Characters')
  .then(accountCharacters => {
    xmlClient.setCharacterID(accountCharacters.result.rowset.row[0].characterID);
    // As xmlClient.get returns an instance of the ES6 Promise object
    // we have the methods all and race at our disposal
    return Promise.all([
        xmlClient.get('char/AccountBalance'),
        xmlClient.get('char/WalletTransactions'),
        xmlClient.get('char/WalletJournal')
      ]);
  })
  .then((values) => {
    console.log(values);
    // [{
    //    <char/accountBalance>
    //  },{
    //    <char/WalletTransactions>
    //  },{
    //    <char/WalletJournal>
    //  }]
  })
  .catch(err => {
    console.log(err);
    process.exit(1);
  });
```

___

### Options
- keyID
- vCode
- characterID
- cache

___

### API

#### thera.XmlClient()

```javascript
var xmlClient = new thera.XmlClient({
  keyID: '123456',
  vCode: 'asdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasdasda'
})
```

thera.XmlApi instances expose the following methods, all of which are chainable.


##### xmlClient.get(resourcePath, [queryParams, requestOptions, force]) (Promise)
.get() fetches the chosen resource from the API. The response is parsed into a direct JSON representation of the XML document and available within a subsequently chained then() callback.

E.g: Fetching the account/Character resource.

```javascript
xmlClient.get('account/Characters')
  .then(accountCharacters => {
    // do something with the accountCharacters
  })
```

Arguments
```
- resourcePath   {string}  API path to desired resource (excluding base URL & filetype suffix). E.g. 'account/Character'
- queryParams    {object}  [optional] Additional query params which will be included in the HTTP request
- requestOptions {object}  [optional] options which will be passed through to request.js .get() method.
- force          {boolean} [optional] if true the HTTP reqeuest will always be made, even if there is already a cached document for the given path.
```


##### xmlClient.setCharacterID(characterID)
Sets the characterID to be used for all subsequent requests to the API.

E.g: setting the character ID to the first character associated with the account & making a request for that characters account balance resource.

```javascript
xmlClient.get('account/Characters')
  .then(accountCharacters => {
    var charID = accountCharacters.result.rowset.row[0].characterID;
    return xmlClient.setCharacterID(charID).get('char/AccountBalance');
  })
  .then(accountBalance => {
    // do something with account balance resource
  });
```


##### xmlClient.setKeyID(keyID)
Sets the Key ID to be used for all subsequent requests to the API.


##### xmlClient.setVCode(vCode)
Sets the Verification Code to be used for all subsequent requests to the API.


##### xmlClient.setConfig(config)
Sets multiple configuration options at once (see options section above).


##### xmlClient.cache
The cache is a key-value store which exposes the methods below.  
If passing your own cache into a thera.XmlClient instance it must implement these methods to be compatible:


##### xmlClient.cache.put(key, document, timeout)  
Returns a promise which resolves once `document` is added to the store indexed by `key`.  
If `timeout` is provided, the document & key will be destroyed after this time.  

Thera uses the endpoint Paths as `key`. for example: 'char/Blueprints'

Example:  
```javascript
xmlClient.cache.put('mykey', {foo: 'bar'}, 1000).then(() => {
  // do something after the document's been added
});
```

##### xmlClient.cache.get(key)
Returns a promise which resolves with the document stored against `key` from the cache

Example:  
```javascript
xmlClient.cache.get('mykey').then(document => {
  // do something with `document`
});
```

##### xmlClient.cache.exists(key)
Returns true if a document exists for `key`.

Example:  
```javascript
xmlClient.cache.exists('mykey').then(exists => {
  if (exists) {
    // do something when mykey exists
  }
});
```

___

### Error Handling
Any Errors encountered during execution of the promise chain can be handled within the .catch() callback:

```javascript
new thera.XmlClient().get('api/CallList')
  .then(...)
  .catch((err) => {
    // err.errorType
    // err.error
    // err.statusCode
  });
```

##### REQUEST_ERR
Rejections with a REQUEST_ERR type are due to the request failing entirely. The error thrown by request.js will be available in err.error.

```javascript
{
  errorType: 'REQUEST_ERR',
  error: <propagated error>
}
```

##### HTTP_ERR
Rejections with an HTTP_ERR type are due to the reciept of a non 200 status code.

```javascript
{
  errorType: 'HTTP_ERR',
  error: <HTTP status code>, }
}
```

##### XML_PARSE_ERR
Rejections with an XML_PARSE_ERR type are due to xml2js erroring. The error thrown by xml2js will be available in err.error.

```javascript
{
  errorType: 'XML_PARSE_ERR',
  error: <propagated error>
}
```

##### INVALID_RESPONSE
Rejections with an INVALID_RESPONSE type are due to the response being in an unexpected structure.


### Develop/Contribute
If you feel you can contribute/help out in any way with the development of Thera, feel free to make a pull request to branch `develop`. Useful contributions at the moment might be: More (persistent) cache classes, journal walking, increasing test coverage.

#### Tests
Mocha/Chai are used for tests, they can be run using the gulp task:
```bash
gulp test
```

Coverage report can be generated using istanbul:
```bash
gulp coverage
```

#### Build
To build the project using Babel:
```bash
gulp build
```

___

### License

The MIT License (MIT)  
Copyright (c) 2016 Simon Tweed

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
