# ctask

Cluster task scheduler for nodejs

## Quick Examples

```javascript
var Job = require('ctask').Job;

new Job({
    name: 'my-first-task',
    interval: 1000,
    action: function (job, callback) {
        // do something
        callback();
  	}
}).start();
```

## Installation

    npm install ctask

## Basic usage

### Parameters

* name - Name of the task, must be unique for each task
* interval - Repeat interval in ms
* shared - Use concurrent workers for task processing
* multi - Can run multiple instances of tasks simultaneously?
* context - Context of target function
* locking - Described bellow
* action - The function that will be executed

Number of concurrent tasks is always equal to number of spawned workers. You can spawn
and destroy workers as you wish. If you destroy all workers, the task with "shared" enabled will
not run.

If you set "shared" to false, the task will be executed only in master process.

### Locking

It is sometimes nessesary to control execution of concurrent tasks. If you enable locking, 
no new concurrent task will be executed until you unlock it or the current one ends.

```javascript
var cluster = require('cluster');

var Scheduler = require('ctask').Scheduler;
var Job = require('ctask').Job;

Scheduler.on('status', function (msg) {
	console.log(msg);
});

Scheduler.on('error', function (msg) {
	console.log(msg);
});

if (cluster.isMaster) {
    cluster.fork();
    cluster.fork();
    cluster.fork();
    cluster.fork();
}

new Job({
    name: 'my-first-task',
    interval: 100,
	shared: true,
	multi: true,
	context: this,
	locking: true,
    action: function (job, callback) {
        setTimeout(function() {
            job.unlock();
            setTimeout(function() {
                callback()
            }, 5000);
        }, 1000);
  	}
}).start();
```

In this example, ctask will execute one task per second up to 4 concurrent tasks 
(depend on number of workers spawned). Each task will end after 5 seconds and its
worker will be released to perform a new task.

### Controling tasks

```javascript
job.start();
job.stop();
job.enabled = false;
Scheduler.enabled = false;
```

## TODO

* Enable user to limit number of concurrent tasks

## License

Copyright (c) 2013 Patrik Simek

The MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
