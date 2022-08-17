# Overview
Describe and understand the NodeJS event loop

# Key Concepts
* The nature of JavaScript is single threaded as of 2020
  * Most applications are run in a single threaded manner
  * but there are additions to the spec that add multithreaded concepts such as `Atomics`, `SharedArrayBuffer` which have not gained wide adoption currently 
  * NodeJS itself is multithreaded with lower level code in C++, third party tools like libuv, and the V8 engine
* JavaScript handles *concurrency* by using the event loop
* The event loop can be thought of as an infinite running loop that continually checks for work to perform
* When work is found, the event loop executes operations with a new *call stack* until work is finished
* The event loop gets to check for more work to do only once a stack is complete
* The concept of *enqueuing* work for a later call stack is important
* The event loop takes a nonzero amount of time to...
  * check for more work to perform
  * prepare a new call stack to perform the work
  * perform work as functions can take a long time to run. A function that takes a very long time to run is considered **blocking the event loop**
* Two call stacks won't exist at the same time due to the single threaded nature of JS

# Academia
* NodeJS utilizes the Continuation-Passing Style (CPS)
* Callbacks, functions that are passed around and invoked by the event loop once a task is complete, drive the CPS pattern
* Callbacks and the event loop are used to allow events in lower level C++ land to cross the boundary and run code in JavaScript
* Asynchronous: Functions that are invoked in the future with a new stack
* Synchronous: Function calls another function in the same stack

# The Event Loop
Note: The event loop implementation differs slightly from browser to NodeJS. NodeJS event loop is optimized for server usage.  
Quickly: The event loop manages a *queue* of events that are used to trigger *callbacks* and move the application along.
* The event loop is made up of distinct phases
* Each phase maintains a queue of callbacks to be executed
* Callbacks are placed in different phases based on how they are used by the application

1. **Poll**
  * executes I/O related callbacks
  * application code most likely executes here
  * main app code starts here
2. **Check**
  * Callbacks triggered via `setImmediate()` run here
3. **Close**
  * Callbacks triggered via `EventEmitter close` events run here
4. **Timers**
  * Callbacks triggered via `setTimeout()` and `setInterval()` run here
5. **Pending**
  * System events are run here

**Special**
* Two special microtask queues can have callbacks added to them while any phase is running
  * First Queue handles callbacks registered with `process.nextTick()`
  * Second Queue handles callbacks related with `promise.reject()` and `promise.resolve()`
* The microtask queues take priority before the next phase

**Bring it all together**
- When an application runs, the event loop is started and the phases are handled one at a time
- Executing code adds callbacks to different queues as the application runs
- When the event loop reaches a phase, all callbacks in that queue are run 
- When all the callbacks in a phase are finished, the microtask queues are checked and worked on
- When all the callbacks in the microtask queues are finished, the event loop moves on to the next phase
> For backend nodeJS application devs, the most important phases to consider are: poll, check, timers and the microtasks after every phase
- When the application runs out of things to do but is waiting for I/O, it will wait in the poll phase
