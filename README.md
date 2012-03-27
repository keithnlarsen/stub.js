Stub.js
=======

# What is Stub.js?

Stub.js is a dead simple to use mocking / stubbing library to help you unit test your javascript nodejs modules.

## Why a stubbing library?

Well, javascript has the ability to mock an object built into it, you can just define an
object literal, and wammo, you have a mock.  But what if I want to track what happened to that
mock when it's passed into some code module that I want to test?  You need something to record
what has happened while it was in use.  That is what stub does, it will record everything that happens
to a stubbed method, and allow you to assert on expected events later.

## But there are already libraries out there for javascript?

Yes, there are, but they all seemed so heavy to me, and tried to take on an almost Java-ish feel
to them.  But this is javascript, why does it have to be so complex?  So I created my own library
that I think is just complex enough to get the job done.

# Contribute

Feel free to submit your comments / feedback / suggestions / feature request.  Just keep in mind, I want this to stay simple to use, and small and light.

# Features

 * stub asynchronous functions
 * stub synchronous functions
 * stub a nested chain of stubs
 * throw simulated errors from within the stub

# Installation

    npm install stub-js

## Dependancies

So far this is dependant on:

  * Nodejs >= 0.4.1
  * I personally use this with should.js and mocha. They ROCK!

# Example with both Synchronous and Asynchronous stubs

### Here is the module we want to isolate and test

```javascript
module.exports = (function() {
  var model;

  function init ( personModel ) {
    model = personModel;
  }

  function findById(id, callback){
    model.find( {_id: id } ).exec( function( err, person ) {
      callback(err, person);
    } );
  }

  return {
    init: init,
    findById: findById
  }
}());
```

### Here is what our test case would look like using Mocha, Should.js, and Stub.js

```javascript
describe( 'myApp.tests.unit.controllers.person', function() {

  var stub = require('stub.js');
  var mockPersonModel;

  var expectedPerson = {
    firstName: 'test',
    lastName: 'test2'
  };

  beforeEach(done){
    // Here we define our mock
    mockPersonModel = {
      // Here we are defining a stub method that will return synchronously a nested mock object.
      find: stub.sync( {
        // Here we are defining an exec method that will execute asynchronously and pass a null
        // and a 'testPerson' object to its callback.
        exec: stub.async(null, expectedPerson)
      } )
    };
    done();
  }

  describe( '.findById(id)', function() {
    it( 'should call the model find function and return the specified object', function( done ) {

      // Now we can test the controller giving it our mock object
      personController.init(mockPersonModel);

      personController.findById('1234', function ( err, actualPerson) {
        actualPerson.firstName.should.equal(expectedPerson.firstName);
        actualPerson.lastName.should.equal(expectedPerson.lastName);
        mockPersonModel.find.called.withArguments( {_id: '1234'} );
        // Heres how we test our nested mock object, that it was called as we expected.
        mockPersonModel.find().exec.called.withNoArguments();
        done(err);
      });
    });
  });
});
```

# API

## Defining a Mock and Stubs

### stub.sync( Error )
If a ```new Error('some error')``` or any object that inherits from Error is provide, it will simulate throwing an error from within the mock.

```javascript
// Define the mock
var mockObject = {
    method: stub.sync( new Error('some error occurred.'))
};
```

When our stubbed method is called it will throw an exception, instead of returning a value.

```javascript
  // throws an error
  mockObject.method( 'something' );
```

We can also later inspect our mock to see what happened.

```javascript
  // this an actual test assertion it will cause a test failure if it doesn't pass.
  mockObject.method.called.withArguments( 'something' );
```

### stub.sync( returnValue )
If any value or object other than Error is passed in will just be returned as the ```returnValue``` from the stub when it is called.

```javascript
// Define the mock
var mockObject = {
    method: stub.sync( true )
};
```

When our stubbed method is called it will return the hard coded value that we specificied.

```javascript
// returns true
var result = mockObject.method( 'something' );
```

We can also later inspect our mock to see what happened.

```javascript
// this an actual test assertion it will cause a test failure if it doesn't pass.
mockObject.method.called.withArguments( 'something' );
```

### stub.async( param1, param2, […])

This will create an `asynchronous` stub that will pass the provided values to a call back.  FYI - The defacto standard for asynchronous calls in javascript is that the first parameter in the callback is reserved for errors, subsequent parameters are for success.

```javascript
// Define our mock and stub
var mockObject = {
    method: stub.async( null, 'our value' )
}
```

When called our stubbed method will call the callback with the values that we specified.

```javascript
// calls the callback with 'our value'
mockObject.method( 'some value', function (err, value ) {
    // Do stuff
} );
```

Again, we can assert what our stubs where called with.

```javascript
// this an actual test assertion, in this case it would cause a failure
// because the stub was actually called with 'some value'.
mockObject.method.called.withArguments( 'something' );
```

## Asserting your Stubs

### mockObject.stub.called

This contains the information recorded while in use.

### mockObject.stub.called.count(2)

It will test if the method was called the number of specified times.

### mockObject.stub.called.withArguments( args…)

It will test if the arguments you passed in matches the actuals that it was called with.

### mockObject.stub.called.withAnyArguments()

It will test if method was called with anything.

### mockObject.stub.called.withNoArguments()

It will test if the method was called without any arguments.

### mockObject.stub.called.time(1).withArguments( args…)

It will test if the first time the method was called that the actual parameters passed to it match what you specified.

### mockObject.stub.called.time(1).withAnyArguments()

It will test if the first time the method was called that it was called with any arguments.

### mockObject.stub.called.time(1).withNoArguments()

It will test if the first time the method was called that it was called without any arguments.

## Asserting Chained Stubs

Assuming a mock / stubs defined in a nested chain like the following:

```javascript
var mockObject {
    stub1: stub.sync( {
        stub2: stub.sync()
    })
}
```

### mockObject.stub1.called.count(1)

It will assert the first call in the chain was called 1 time.

### mockObject.stub1().stub2.called.count(1)

It will assert the second call in the chain was called 1 time.

## License

(The MIT License)

Copyright (c) 2011 Keith Larsen

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.