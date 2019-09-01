## Results

Check out the live scores [here](https://veryjos.com/pinball)!



This stylish, era-appropriate webpage was designed to render in IE5.

## Header

This weekend, me and my friend Brandon worked together to disassemble **Space Cadet Pinball** on Windows 98.

I worked on all the native pieces, so reverse engineering the binary and writing a small program to bootstrap the UI. Brandon worked on the backend to keep all the scores, as well as the web-based frontend- which is no small feat.

Creating a SPA compliant with **Internet Explorer 5** is no small feat.

## Reverse Engineering the Binary

Ultimately, to add a new high score list, there are three locations that I need to hook:
 - Program launch -> Inject DLL
 - "View Highscores" menu option -> Display custom frontend
 - Game over -> Steal score out of RAM, submit score to backend

Finding the first one is easy. Just pause the debugger on attach, and find a nice spot to call your injected subroutine.



Fairly straightforward! We've lazily stuck our path to the DLL in ascii on top, perhaps unsafely, and right below we have our code cave for actually loading the DLL.



This next bit is calling the function **LoadLibraryA**, which will load and inject the DLL we've previously prepared.

**LoadLibraryA** expects to be called with the **cdecl calling convention**. That means that we must push the path to the our load onto the stack before executing the function.



After we do that, we can use **GetProcAddress** to get pointers to functions that we wrote and exposed in our DLL. This makes it significantly easier to inject complex behavior, since we no longer have to write assembly by hand.

Now that we have the ability to **inject a custom DLL**, it's time to *create* the DLL.

## Creating the DLL

We're going to use **multiprocessing** to simplify the approach.

We'll create **two separate binaries**:

  - The injected DLL
  - A **standalone viewer** for the high scores

It's best to keep the injected DLL as **simple as possible**. The DLL will only launch the viewer, and **block the game thread** until the viewer is closed. An **optional argument** over stdin will signal that a new score is being submitted.

To facilitate this, all we have to put in our injected DLL is this one command:



## Creating the Viewer Binary

To keep things quick and easy, we decided to build a **web-based frontend** instead of something native. This means that our viewer binary will embed an **ActiveX MSIE control**, and all of the business logic will be handled in JavaScript.

When I say quick and easy, I mean this actually took a few hours and a lot of headaches. Documentation for ActiveX in pure WinAPI and C is nearly non-existant in 2018, but I eventually clobbered together enough garbage to get something working.



With the viewer binary and DLL done, it's now time to inject some assembly in the right place in order to bootstrap these two binaries.

## Hooking "View Highscores"

The next step is to display the frontend when the "**View Highscores**" menu option is clicked. First, we have to locate the subroutine that's invoked when the user clicks the button.



Luckily, the "View Highscores" dialog enters its **own blocking event loop**. This means that all we have to do is pause the program while the high-scores list is up, and look up the stack.



We find a call to **ShowDialog**, which we nuke into a trampoline to invoke the function we expose in our DLL. The 0 passed to our subroutine indicates that we have no score to submit, and we only want to view.

That lets us jump into DLL-land, which in-turn lets us block the game thread while we display the high score list!

## Hooking "Game Over"

The last step is to **submit the player's score** when they lose-- so naturally, we need to be able to find the subroutine that runs when the player loses. 

We can use the same method as last time. Because we know the game will enter a totally separate event loop when the high-score list is being displayed, we can just lose, view the high score list, and then look up the stack to find how we got there.



A quick inspection of the stack leads us here. Setting a breakpoint and stepping through reveals that the score is being written to the **ESI register**. All we have to do is stick a call to a subroutine that invokes the function in our DLL, and then escapes out of the "Game Over" state.

## The Finished Product

At this point, we're done! Here's what it looks like to earn a high-score:

<figure>
  <iframe src="https://gfycat.com/ifr/highlevelelementaryglassfrog" frameborder="0" scrolling="no" allowfullscreen="allowfullscreen" />
</figure>

You can view the high-score list here: [https://veryjos.com/pinball](https://veryjos.com/pinball)

Thanks for reading :)
