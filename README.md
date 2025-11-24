# Omnis Promises

## Installation
The `lib` folder contains a `promises` Omnis library (exported to JSON), which you should import into Omnis, and exposes functionality for simple Promise-like behaviour in Omnis Studio.

Note that the Objects used here should generally be **Object References**.


## What is a Promise?
A Promise represents a pending action (usually asynchronous), which will complete at some point in the future. Promises are either '**resolved**' upon success, or '**rejected**' on failure.

The `promises` library contains an `oPromise` Object, representing a Promise.

The intended way to use these is to return a promise from a method which runs some asynchronous action.
The *caller* can then call `$then()` on this promise, passing an Item Reference to a method to execute when the promise 'resolves'.

This can simplify your code when dealing with asynchronous actions, so you can do things like:

```Do iHTTP.$fetch(lURL).$then($cinst().$processResults.$ref)```


## Promise scope
As promises usually represent asynchronous actions, it's important that you do not let them go out of scope, as they could be destroyed.

The `oPromiseManager` Object class is provided as an effective means of automatically managing Promises' lifecycles.

Rather than create instances of oPromise directly, instead create a single instance var of *oPromiseManager* on any class which may create promises, and call its `$createPromise()` method to create your promises.
*oPromiseManager* will then keep a reference to your promise until it completes, so callers do not need to be concerned about keeping the object reference alive.


## oPromise Methods

### **$then(pMethodRef, [pExtraParams])**: 
Specify the method to call when the promise completes.
This will also be called if the promise is rejected, **if** you have not assigned a method to **$catch**.

The method's signature should match [that expected for promises](#promise-method-signatures).

You only use this when you are *consuming* promises.

**Parameters:**
- **pMethodRef** (Item Ref): An *Item Reference* to a method of an instance: that method will be called when the promise completes.
- (Optional) **pExtraParams** (Row): A Row containing extra data you wish to pass through to the method.

e.g:
```
Do lPromise.$then($cinst().$myMethod.$ref)
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
Sets up the promise as a very simple $callbackinst for the given worker object, and will be resolved, with the worker's $completed method's result row.

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

