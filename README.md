#Throttle

Throttle is a function that you can call at any rapidity, but which forwards that call at certain predetermined intervals. Typical use of throttle is to tame scroll and resize events, bot that are fired at very rapid pace by browser. When you need to do some time consuming things ( lots of DOM manipulations, etc ) then such events choke the browser and things will get ugly. The usual case against that is to write a throttle and call the callback at, lets say, every `250ms`. It is true that callbacks for such events should use `requestAnimationFrame()`, but that is often not good enough. Better to use both - throttle and then call callbacks with `requestAnimationFrame()`. 

## Why?

The problem with most throttle implementations around is that they create a lot of garbage. Somewhere in the throttle you need to set timeout and call the callback, usually this is done like this:

	var args = Array.prototype.slice.call(arguments);
    setTimeout(function () {
     	callback.apply(context, args);
    }, delay);
    
In essence, throttles role is to called a __lot__ and hence the throttle also creates garbage a __lot__. Here at every call we access magical array of `arguments` that is created at request and must be garbage collected later, also we create new anonymous function, which also is needed to garbage collect.

Also, in special case we need to manage how the callbacks are called and differentiate between calling phases. The different phases are

1. first call to activate throttling (or f.e scrolling started)
2. throttling ( or f.e. scrolling is ongoing )
3. last call, throttling deactivated (or f.e. scrolling ended)

I had need for all those phases in various projects. Rarely for first call, mostly to react only to last call. The latter option (only fire at last) allows usage of this throttle for example:

- to implement live search - add throttle to input fields keyup event and fire ajax call at throttle's last call. 
- to implement idle detector - at simplest, add mousemove listener to document, set interval to 5s. At last call you have entered into idle state, at first call you are busy again.

So those were goals for this implementation - separate and configurable phases and no garbage. 

The need of not using `arguments` has one limitation - the amount of arguments callback function has can not be arbitrary. I did choose 3, since i never ever had need for more than 1.

The throttle is initiated from class and closed over. All `setTimeout()` callbacks are created as class methods and bound to class with general pattern like this:

	// in constructor 
	this.run = this.run.bind(this);
	// and later
	setTimeout(this.run, 100);
	

## Usage

	// create throttle
	var thr = throttle(interval, fn, context, nop, nod)
	
	// call throttle
	thr('foo')
	

Where:  
__interval__ is the delay you wanted callback is called, in milliseconds  
__fn__ is the callback, 3 parameters and calling phase are passed to callback, phase ranges from 1..3

	// abstract callback
	var callback = function(a1, a2, a3, phase) {
		console.log(phase, a1);
	}
	

__context__ is optional context at which callback is called, default is window  
__first__ whether to call callback at first throttle call, default is false   
__after__ whether to call callback only after last call to throttle, default is false  



Examples:

Add `250ms` throttle to resize event

	window.addEventListener('resize', throttle(250, function(event) {
		// do stuff 
	}), false);



## License

Copyright (c) 2014 Priit Pirita, released under the MIT license.


