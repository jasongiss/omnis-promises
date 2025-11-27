# Omnis Promises

## Installation
The `lib` folder contains a `promises` Omnis library (exported to JSON), which you should import into Omnis, and exposes functionality for simple Promise-like behaviour in Omnis Studio.
Note that to import this library, you need to use Studio 12, and make sure that the **$exportimportjsonoptions** Preference has **fullexportimport** set to **false**.

Note that the Objects used here should generally be **Object References**.

There is also a `promiseExample` library in the `lib` folder. You can also import this into Omnis Studio to see a simple example of using promises.


## What is a Promise?
A Promise represents a pending action (usually asynchronous), which will complete at some point in the future. Promises are either '**resolved**' upon success, or '**rejected**' on failure.

The `promises` library contains an `oPromise` Object, representing a Promise.

The intended way to use these is to return a promise from a method which runs some asynchronous action.
The *caller* can then call `$then()` on this promise, passing an Item Reference to a method to execute when the promise 'resolves'.

Promises can also be **chained**, by returning promises from callback methods. `$then()` returns a new chained promise, which you can call `$then()` on again to chain the next action, and so on.

When you have a Promise Chain, if a promise is rejected, it will follow down the promise chain to fire the first rejection handler it finds.

This can simplify your code when dealing with asynchronous actions, so you can do things like:

```
Do iHTTP.$fetch(lURL) Returns lPromise
Do lPromise.$then($cinst().$processResults.$ref) Returns lPromise  ## When $fetch completes, call $processResults
Do lPromise.$then($cinst().$updateDB.$ref) Returns lPromise  ## When $processResults completes, call $updateDB
Do lPromise.$catch($cinst().$handleError.$ref)  ## If an error occurs anywhere along the promise chain, call $handleError

# Or even do this all on one line:
Do iHTTP.$fetch(lURL).$then($cinst().$processResults.$ref).$then($cinst().$updateDB.$ref).$catch($cinst().$handleError.$ref)
```


## Promise scope
As promises usually represent asynchronous actions, it's important that you do not let them go out of scope, as they could be destroyed.

The `oPromiseManager` Object class is provided as an effective means of automatically managing Promises' lifecycles.

Rather than create instances of oPromise directly, instead create a single instance var of *oPromiseManager* on any class which may create promises, and call its `$createPromise()` method to create your promises.
*oPromiseManager* will then keep a reference to your promise until it completes, so callers do not need to be concerned about keeping the object reference alive.


## oPromise Methods

### **$then(pFulfilledMethodRef, [pRejectedMethodRef, pExtraParams])**: 
Specify a method to call when the promise completes successfully ('fulfilled'), and optionally another to call if it fails ('rejected').

If no rejected method is provided here (**or further down the promise chain**), *pFulfilledMethodRef* will be called on failure too.

The methods' signatures should match [that expected for promises](#promise-method-signatures).

You only use this when you are *consuming* promises.

**Parameters:**
- **pFulfilledMethodRef** (Item Ref): An *Item Reference* to a method of an instance: that method will be called when the promise completes successfully.
- (Optional) **pRejectedMethodRef** (Item Ref): An *Item Reference* to a method of an instance: that method will be called when the promise fails. (Or any previous failure in the promise chain which has not been handled)
- (Optional) **pExtraParams** (Row): A Row containing extra data you wish to pass through to the callback methods.

**Returns:**
- A new promise, representing the 'next' promise in the chain. You can chain further $then/$catch calls on this.

e.g:
```
Do lPromise.$then($cinst().$myMethod.$ref) Returns lNextPromise
```

*(Note the syntax to get a reference to an instance's method: it includes parentheses after the inst, to force the notation to evaluate at this point)*


### **$catch(pMethodRef, [pExtraParams])**: 
Specify a separate method to be called when the promise is **rejected**.

The method's signature should match [that expected for promises](#promise-method-signatures).

You only use this when you are *consuming* promises.

**Parameters:**
- **pMethodRef** (Item Ref): An *Item Reference* to a method of an instance: that method will be called when the promise is rejected.
- (Optional) **pExtraParams** (Row): A Row containing extra data you wish to pass through to the method.

e.g:
```
Do lPromise.$catch($cinst().$myFailureMethod.$ref)
```


### **$resolve(pResults)**: 
Mark the promise as completed **successfully**, and pass some results. 

You only use this when you are *producing* promises.

**Parameters:**
- **pResults** (Row): A row of results to pass to your consumers.

e.g:
```
Do lPromise.$resolve(lResults)
```


### **$reject(pResults, pErrorText)**: 
Mark the promise as **failed**, and pass some results, with an error description. 

You only use this when you are *producing* promises.

**Parameters:**
- **pResults** (Row): A row of results to pass to your consumers.
- **pErrorText** (Character): A description of the error.

e.g:
```
Do lPromise.$reject(lResults,"No username was provided")
```


### **$attachToWorker(pWorkerRef)**: 
Sets up the promise as a very simple $callbackinst for the given worker object, and will be resolved, with the worker's $completed method's result row, when the worker completes.

For **very simple** scenarios, this is a helpful shortcut. But note that it will never reject the promise.

You only use this when you are *consuming* promises.

**Parameters:**
- **pResults** (Row): A row of results to pass to your consumers.
- **pErrorText** (Character): A description of the error.

e.g:
```
Do lPromise.$reject(lResults,"No username was provided")
```



## Promise method signatures
The methods which you assign to *$then* or *$catch* receive the following parameters:

1) **pResults** (Row): A row containing the results of the action.
2) **pExtraParams** (Row): The pExtraParams you passed through when assigning your *$then* or *$catch* method.
3) **pErrorText** (Character): Any error text (if the promise was rejected). It will never be set if you use a separate *$catch* method.
4) **pSuccess** (Boolean): True if the promise was resolved successfully. It will never be false if you use a separate *$catch* method.

