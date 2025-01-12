﻿# BlockNot Arduino Library  
  
**This library enables you to create non-blocking timers using simple, common sense terms which simplifies the reading and writing of your code. It offers, among several things, convenient timer functionality, but most of all ... it gets you away from blocking methods - like delay() - as a means of managing events in your code. Non-Blocking is the proper way to implement timing events in Arduino code and BlockNot makes it easy!**  
  
## Getting started immediately
Here is an example of BlockNot's easiest and most common usage: 
  
```C++  
#include <BlockNot.h>  

BlockNot myTimer(1300);  
  
void loop() {  
	if (myTimer.TRIGGERED) {  //Runs every 1300 milliseconds
		Serial.println("Hello World!");
	}  
}  
```  
**YES, it's that simple!** But keep reading to learn about other features.

  
## Theory behind BlockNot  
  
This code is ugly, cumbersome and annoying to type all the time. It is the traditional form of non-blocking timers:

```C++
long someDuration = 1300;
long startTime = millis();
if (millis() - startTime >= someDuration) {
    //Code to run after someDuration has passed.
}
```

This, on the other hand, is simple, elegant, and intuitive!
```C++
if (myTimer.TRIGGERED) {
    //Code to run after timer has triggered.
}
```
The idea behind BlockNot is very simple. You create the timer, setting it's duration in a single line of code. Then in your loop or in your other methods, you simply engage the timer to see if your defined time duration has come to pass or not. You can do this by calling the methods directly, or by using some macros added into the library based on simple words that greatly enhance the readability and write-ability of your code. The macros also eliminate the need to pass arguments into the methods which further simplifies your code.

For example, if you wanted to see if the timers duration has come to pass, but you don't want to reset the timer, you can use this method:
```C++
if (myTimer.triggered(NO_RESET)) {}
```
OR, you can do it like this:
```C++
if (myTimer.HAS_TRIGGERED) {}
```
They both do the same thing, but in terms of readability, the second example is the obvious choice. Readability MATTERS, especially as your projects get more complicated.

This is a simple graph showing you how BlockNot timers work. Keep in mind, that what matters here is that from time point 0 until the duration has been reached, your program continues to run - doing other things. delay will not do that for you:

![Graphical illustration of timer in action](https://i.imgur.com/Hxrjhvo.png)

## The Trigger

BlockNot is all about the trigger event. Picture someone standing there holding a gun pointed towards the sky, and they have been instructed to pull the trigger on the gun every 10 seconds.  Now you know what a trigger event is. It is that moment in time when the timers duration has come to pass. You set the duration when you create your timers. You can also change the duration in an existing timer by setting it to a new value, or by adding / subtracting time to the timers current duration. More about that in the method descriptions.

Chances are, you will use this library more often than not, by simply checking the timers trigger, then run some code if the trigger event has transpired.
```C++
if (voltageReadTimer.TRIGGERED) {
	readVoltage();
}
```
## The Reset

Resetting a timer is critical to performing repeated events at the right intervals. Using the usual method of implementing non-blocking timers, you would first set a variable to the current millis()
```C++
long startTime = millis();
```
Then in your loop you would subtract the startTime from millis() and if that difference was equal to or greater than your desired time duration, you would run your code... However, if you need that behavior to repeat at the same intervals of time, you then have to reset startTime to the current millis(); Your code would look something like this:
```C++
long duration = 1300;
if (millis() - startTime >= duration) { 
	//run code 
	startTime = millis();
}
```
##BlockNot does all of that for you
- depending on which method you use to query it. For example when you do this
```C++
if (myTimer.TRIGGERED) { //my code }
```
the resetting of startTime is automatic. Whereas if you do this
```C++
if (myTimer.HAS_TRIGGERED) { //my code } 
```
the startTime **does not reset** and that test will always come back true every time it is executed in your code, as long as the timers duration has passed.

There is a method which I have found quite handy in specific situations. And that is the FIRST_TRIGGER method. You use it like this:
```C++
if (myTimer.FIRST_TRIGGER) { //my code }
```
That method will return true ONE TIME after the timers duration has passed, but subsequent calls to that method will return false until you manually reset the timer like this:
```C++
myTimer.RESET;
```
The simplicity of that kind of method and what it allows you to do in a single line of code cannot be over stated. Using **one line of code**, you can do a single trigger event consisting of anything at all!

Why would you need to do that? There are countless scenarios where that would be immediately useful. For example, I've used it with stepper motor projects where I want an idle stepper motor to be completely cut off from any voltage when it's been idle for a defined length of time ... lets say 25 seconds. 

So first, we define the timer
```C++
BlockNot stepperSleepTimer (25000); // 25 seconds
```
Then in the code that we use to cause a stepper to actually turn, we add into it the resetting of this sleep timer like this:
```C++
stepperSleepTimer.RESET;
```
Then, in your loop, you would put something like this:
```C++
if (stepperSleepTimer.ONE_TRIGGER) {
	sleepStepper();
}
```
So that if the stepper hasn't moved in the last 25 seconds, it will be put to sleep and that sleep routine won't be constantly run over and over again. Yet when the stepper is engaged again, then that sleep timer gets reset.

BlockNot makes it easy for you to establish single trigger events, without ever needing to rely on the Delay method. Spend your efforts working on the core of your project and let BlockNot handle your timing needs.

##Enable / Disable
In the first update to BlockNot, the ability to disable and enable a timer was introduced. I found myself in a situation where I realized disabling a timer would be more efficient than declaring a boolean variable and relying on its value before checking a timer.

Before I show an example of when you might want to disable a timer, Let's describe what you can expect when engaging the timer if it is in a disabled state.

When a timer is disabled, any call to the timer that would return a boolean value will ALWAYS return false. And when you query the timer where a numeric value is supposed to be returned, it will return a ZERO by default, although you can change what number it returns for those methods, as long as the number you set is a positive number (BlockNot does not ever deal with negative numbers, since time cannot go backwards),

These methods will ALWAYS return false when a timer is disabled (for macro's see the end of this document):

*    **triggered()**
*    **notTriggered()**
*    **firstTrigger()**

These methods will return a ZERO by default (when a timer is disabled), or whichever value you chose to get back (explained below).
*    **getDuration()**
*    **getTimeUntilTrigger()**
*    **timeSinceLastReset()**

These methods will do NOTHING if the timer is disabled. Meaning that the changes you wish to make to the timer simply cannot be done if the timer is disabled.
*    **setDuration()**
*    **addTime()**
*    **takeTime()**
*    **reset()**

###How do I chose what number is returned on a disabled timer?
It's EASY PEASY! All you do is add your desired number when you create your timer like this
```C++
BlockNot myTimer(2000, 8675309);
```
The first number is the timers duration and the second number is the return value for all methods that return numeric values when the timer is disabled (Jenny's phone number in the example ;-). That second number CANNOT be a negative number and the compiler will throw an error if you try to set it to one.

You can disable / enable a timer using these methods (for macros, see the end of this document).
```C++
myTimer.enable();
myTimer.disable();
```

And you can find out if the timer is enabled or disabled like this:
```C++
myTimer.isEnabled();
```

You can also flip the state of the timer (if disabled, it will enable it, or if it's enabled, it will disable it):
```C++
myTimer.negateState();
```
But the macro's are far easier and more intuitive:
```C++
myTimer.FLIP;
myTimer.SWAP;
myTimer.FLIP_STATE;
myTimer.SWAP_STATE;
```
Why would you want to just change the state with one line of code? I can simply disable or enable at will, right?

ABSOLUTELY! However, let's say that you have an event that fires at a regular interval of time, that you engage by using the reliable TRIGGERED method in an if statement. It might be useful to allow the user to make that timed event stop or start at will by pressing a button.

```C++
if (BUTTON_PRESSED) {
    myTimer.FLIP;
    delay(350); //de bouncing the button - delay still has its uses now and then :-)
}
```

So when the timer is disabled by pressing the button, it will always evaluate to FALSE when you call the TRIGGERED method so that it simply won't execute the code that you placed in the if statement. This is much easier than setting up a separate boolean variable and tracking it with the button press.

## Summary

Well, those are the fundamentals of BlockNot. 

Simple, right?

BlockNot is a library intended to make the employment of non-blocking timers easy, intuitive, natural and obvious. It can be engaged with simple single word method calls or by calling the core methods directly.

There are more methods that allow you to affect change on your timers after instantiation and also methods to get info about your timers. You can change the duration of an existing timer in three different ways, you can reset the timer, or you can even find out how much time is left before the trigger event occurs, or find out how much time has passed since the timer was last reset.

## Examples

There are currently three examples in the library.

#### BlockNotBlink
This sketch does the same thing as the famous blink sketch, only it does it with BlockNot elegance and style.


#### BlockNotBlinkParty
If you have a nano or an uno or equivalent laying around and four LEDs and some resistors, connect them to pins 9 - 12 and run this sketch. You will immediately see the benefit of non-blocking timers. You could never write a sketch that could do the same thing using the delay() command.  It would be impossible.

#### Non-Blocking MATTERS!

#### TimersRules
This sketch has SIX timers created and running at the same time. There are various things happening at the trigger event of each timer. The expected behavior is explained in the out Strings to Serial. Read them, then let it run for a minute or so then stop your Serial monitor and look at the output. You should be able to look at the number of seconds that is given in each output, and compare the differences with the expected behavior and see that everything runs as it is expected to run.<BR><BR>For example, when LiteTimer triggers, you should soon after that see the output from stopAfterThreeTimer.  When you look at the number of seconds in each of their outputs, you can see that indeed it does trigger three seconds after being reset, but then it does not re-trigger until after it is reset again.
<BR><BR>These examples barely scratch the surface of what you can accomplish with BlockNot.

# Methods

Below you will find the name of each method in the library and any arguments that it accepts. Below that list, you will find the names of the macros that are connected to each method along with the arguments that a given macro may or may not pass to the method. The macros are key to making your code simple. BlockNot is key to making non-blocking timers easy and practical.

#### For any method call that resets a timer by default, the resetting behavior can be overridden by passing **NO_RESET** into the methods argument.


*    **triggered()** - Returns true if the duration time has passed. Also resets the timer (override by passing NO_RESET as an argument).
*    **notTriggered()** - Returns true if the trigger event has not happened yet.
*    **getDuration()** - Returns an unsigned long, the duration that is currently set in the timer.
*    **setDuration()** - Override the current timer duration and set it to a new value. This also resets the timer. If you need the timer to NOT reset, then pass arguments like this (newDuration, NO_RESET);
*    **addTime()** - Adds the time you pass into the argument to the current duration value. This does NOT reset the timer. To also reset the timer, call the method like this **addTime(newTime, WITH_RESET);**
*    **takeTime()** - The opposite effect of addTime(), same deal if you want to also reset the timer.
*    **getTimeUntilTrigger()** - Returns an unsigned long with the number of milliseconds remaining until the trigger event happens.
*    **timeSinceLastReset()** - Returns an unsigned long with the number of milliseconds that have passed since the timer was last reset or instantiated.
*    **reset()** - Sets the start time of the timer to the current millis().
*    **firstTrigger()** - Returns true only once and only after the timer has triggered.
*    **enable()** - enables the timer. Timers are enabled by default.
*    **disable()** - disables the timer.
*    **isEnabled()** - returns true or false depending on the current state of the timer.


Here are the macro terms and the methods that they call along with any arguments they pass into the method:

*    **TIME_PASSED** - timeSinceLastReset() 
*    **TIME_SINCE_RESET** - timeSinceLastReset() 
*    **ELAPSED** - timeSinceLastReset() 
*    **TIME_REMAINING** - getTimeUntilTrigger() 
*    **REMAINING** - getTimeUntilTrigger() 
*    **DURATION** - getDuration() 
*    **DONE** - triggered() 
*    **TRIGGERED** - triggered() 
*    **DONE_NO_RESET** - triggered(NO_RESET) 
*    **HAS_TRIGGERED** - triggered(NO_RESET) 
*    **NOT_DONE** - notTriggered() 
*    **NOT_TRIGGERED** - notTriggered() 
*    **FIRST_TRIGGER** - firstTrigger() 
*    **RESET** - reset() 
*    **ENABLE** - enable()
*    **DISABLE** - disable()
*    **SWAP** - negateState()
*    **FLIP** - negateState()
*    **SWAP_STATE** - negateState()
*    **FLIP_STATE** - negateState()
*    **ENABLED** - isEnabled()

You can, of course create your own macros within your code. So, for example, let's say that you wanted a macro that overrides the default reset behavior in the setDuration() method, which by default, will change the duration of the timer to your new value and will also reset the timer. But lets say you want to change the duration WITHOUT resting the timer and you wanted that to be done with a word that makes more sense to you. 

```C++
#define QUICK_CHANGE(value) myTimer.setDuration(value, false)

QUICK_CHANGE(3200);
```

The only difference here, is that you cannot make a macro that applies universally to all of your timers. You would need to make one for each timer you have created. If you have ideas for macros that you think should be included in BlockNot, let me know and I'll see about adding it.
  
Thank you for your interest in BlockNot. I hope you find it as invaluable in your projects as I have in mine.  
<BR>Mike Sims,<BR>
sims.mike@gmail.com
> Written with [StackEdit](https://stackedit.io/).
