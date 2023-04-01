**CASEY**: I'm certainly happy to end the conversation here if you'd prefer. I appreciate you taking the time to do it, it's been rather lengthy :) But my closing analysis doesn't really line up with yours. So I'll include my response here for the record, but, if you'd rather not keep going, that's fine with me, too.

Essentially, I don't think our differences come down to preferences. I think the problem is that I'm not seeing where the developer cycles are being saved. I understand what you are _intending_ to do when you say that you want to trade CPU cycles for programmer cycles, but I don't see _where the actual savings come in_.

That's what I was trying to get at. Since we only just started talking about architecture, I tried to drill down on "dependency inversion" to see where you're seeing the tradeoff, because that was the one thing that you said was a _positive_ for your architecture style (runtime was the negative). But I don't feel like we ever got to the part where the benefit was made clear.

There was an example about file I/O, but as hopefully I demonstrated with the H/C file example, it does not require any new design, nor even any actual _work_, to simply cut the dependency at the interface boundary _no matter what_ the underlying architecture was. Any set of N function calls to a device driver can have the device driver swapped with another device driver, and now that driver implements those function calls. This is how it worked before OOP, and it's how it still works now in many file I/O subsystems. So I can't really use that to evaluate any "programmer cycle savings" for any specific architecture. The architectural benefits have to be demonstrated somewhere _below_ this cut, because there is not really any architecture I can think of that can't be cut in this way.

Now, thinking about the design of a file I/O subsystem, there certainly _are_ many things that I think are important, both in terms of "CPU cycles" and "programmer cycles". But in order to get into those you'd have to go beyond the basic premise that you want to be able to have read() calls that get fielded by different devices, OSes, or drivers.

If we end the discussion now, we also don't get a chance to talk about your payroll example, which was:

```
    public class Payroll {
      public void doPayroll(Date payDate) {
        for (Employee e : DB.getAllEmployees()) {
          if (isPayDay(e, payDate)) {
            Paycheck check = calculatePaycheck(e, payDate);
            pay(e, check);
          }
        }
      }
    }
```

You listed a bunch of things you wanted the implementation of `isPayDay()`/`calculatePaycheck()`/`pay()` to handle:

* Employees can be salaried, hourly, or commissioned
* Hourly employees are paid weekly
* Commissioned employees are paid twice a month
* Salaried employees are paid monthly
* Hourly employees are paid time and a half for overtime
* Commissioned employees are paid a base salary plus a commission 
* Employees can get their paychecks directly deposited, mailed to their home, or held at the paymaster

That certainly sounds like a well-specified problem to me, so perhaps you can show me what operand-primal design you had in mind for the implementation that provides all these things and "saves developer cycles" over the operation-primal one? Because obviously, the code you wrote above _does not imply either one_. A programmer could do everything from inheritance hierarchies to a giant hard-coded calculatePaycheck function to satisfy your requirements here, so the savings in developer time is presumably coming from the _ease_ with which you can implement the underlying functions, and the corresponding ease with which you can make changes to them. Yes?

So if you _would_ like to elaborate on that design here, we can keep going. Or, if you've got a github where you implemented this example, you can point me to it and I'll go read it.

If not, no worries, we can call it a day.

**Bob**: I'm happy to continue.  I've moved this discussion to file -2.

You raised two topics above.  The saving of programmer cycles, and the utility of dynamic binding.  I think we need to separate these two concerns.  

#### Programmer Cycles.
Regarding programmer cycles vs computer cycles.  Take a look at the file `tdox.c` in this respository.  Perhaps you've seen this before.  If not, if you compile and run it you'll get a cute surprise.

The point, of course, is that there are programming styles that can waste massive amounts of programmer cycles, and there are styles that concerve programmer cycles.  I have a style that I have refined through hard experience over the last 50 years; and I believe saves me cycles.  I recommend it to others based upon that experience and that belief. This style is based upon far more than just the occasional use of dynamic polymorphism.  Rather it focuses on function size, naming heuristics, code organization, dependency management, and quite a few other parameters. 

Can I prove my belief?  Not mathematically; just as I am sure that you cannot mathematically prove that your favorite style saves more or less programmer cycles than mine.  All either of us can do is make _human_ arguments based upon our own particular perceptions of programmer psychology.  If you'd like to go there, we can certainly try.  But I fear we could get caught in a subjectivity trap.

#### Dyamic Binding
So let's first focus on the file I/O example.  Yes, of course, you can cut the dependency at the interface boundary and use link-time late binding to protect the applications from changes in the IO devices.  The switch statement (or if/else chain) that selects the IO device is kept in the OS and the application remains ignorant of it.  However, the OS has to be linked with all the drivers in order for the switch statement to be able to invoke them.  If we want to add a new driver, we need to link it into the OS and modify the switch statement accordingly.  

Let us say, however, that we have a system that is continuously running and we cannot afford to take it off line to re-link the OS whenever a new IO driver is created.  Rather we want compile that new IO driver into a DLL, add it to a special directory, and let the OS hot load it.  That's a use case where dynamic binding comes in pretty handy. 

Do we agree so far?

**CASEY**: Well, if we are just talking about the fact that at various points we want to use a dynamic linker in some way (meaning either we overwrite source code displacements or we patch a function table), then certainly we agree. Any time we want to load something dynamically, we need a dynamic linker, sort-of by definition (unless you are in a completely JIT-ed environment, which is very rare today).

So I don't disagree about that. But what I am trying to get at is which practices make the dynamically-linked system more or less likely to "save programmer cycles". I would argue - very strongly in this case specifically, but in most cases generally - that enums/flags and if/switches are much better than classes for this design. I think they are much better _along every axis_. They will be faster, easier to maintain, easier to read, easier to write, easier to debug, and result in a system that is easier for the user to work with as well.

But my _understanding_ - and feel free to correct me if this entire conversation is based on a misconception - is that you think classes with virtual functions will be the thing that does all of these _except_ perhaps the "faster" part (the aforementioned trade of CPU cycles for programmer cycles). While I agree there isn't a mathematical proof one way or the other, the purpose of a discussion like this is to get at the specifics, so seeing what each person calls a good implementation is very useful even if neither of us changes our minds. At least we _know_ specifically what the other person is talking about in practice, whereas right now I don't really know what the claimed savings is. Once we drill down to it, I may totally disagree that it's a savings, but at least I'll know _exactly_ what you're talking about, whereas at the moment I don't feel like I do.

Since the file example has gotten the most traction thus far, perhaps I can prevail upon you to show me what the design would be that you'd actually propose for something like this? Because the devil is in the details with things like programmer productivity, I'd like to keep it as real-world as possible, so perhaps we say that this is going to be a raw read/write API, so we do not have to include things like file metadata, to keep the scope small for purposes of discussion.

So perhaps we imagine a real modern operating system - say Microsoft Windows or Linux - is providing a new raw IO interface. It needs to:

* Allow the user to specify a particular raw device out of several that might be in the system.
* Allow the user to read n bytes from a specific offset on the device to a user-provided piece of memory
* Allow the user to write n bytes from a user-provided piece of memory to a specific offset on the device

What does this interface look like if designed properly according to your design principles?

**Bob**:  I guess that depends a lot on the language and the application. Java can read bytes from an `InputStream`.  In clojure I'd likely do something simple like slurp in the whole file and navigate in memory to the bytes I need.  But I think you are asking a different question.  I think you want me to continue the story I told about the '70s and the Unix style IO interface.  

I'm not sure it's possible to improve upon that at the lowest levels.  The vtable of `open`, `close`, `read`, `write`, and `seek` are pretty good.  This scheme allows me to write the following program to copy one device to another without knowing what the device is:

	void copy() {
	  int c;
  	  while((c=getchar()) != EOF)
  		putchar(c)
	}

Where `getchar` and `putchar` are helper functions that call `read` and `write` on `stdin` and `stdout` repspectively.  If you compare this code to the kind of code that we had to write in the '60s to do roughly the same thing, the savings in programmer cycles is profound.

Does the above code depend upon dynamic polymorphism?  In UNIX it is certainly implemented that way.  But, as you pointed out earlier, it would be possible to create an OS interface that used link-time binding instead of run-time binding to accomplish the same thing -- with the exception that the IO drivers could not be dynamically loaded.

>_I could stop here and simply say that there are times when the link-time approach is better than the run-time approach; and that's true.  However, you said that enum/flags and if/switches are better in most cases. So let's examine that._

First we need to acknowledge that since the 90s there has been a very large emphasis on dynamic linking.  The creation of DLLs, Jars, and Shared Libraries moved the linker into the loader.  The reason this was considered valuable was so that we could build our applications using a plugin architecture.  You may remember `ActiveX` and DCOM, all that stuff from back then.  It ought to be clear that link-time binding that uses `if/switch` below the interface _impedes_ the plugin strategy; but let me explain why.

In the if/switch case, what does the OS look like?  Each of the five interface functions mentioned above probably has a switch statement in it.  It looks something like this in C(ish) code:

`file read.c`

	#include "devids.h"
	#include "console.h"
	#include "paper_tape.h"
	#include "..."
	#include "..."
	#include "..."
	#include "..."
	#include "..."
	void read(file* f, char* buf, int n) {
		switch(f->id) {
			case CONSOLE: read_console(f, buf, n); break;
			case PAPER_TAPE_READER: read_paper_tape(f, buf n); break;
			case...
			case...
			case...
			case...
			case...
		}
	}
	
This is a fairly ugly source file.  The more devices there are the bigger this file grows both in terms of `#include` and `case` statements.  The number of outgoing dependencies also grows, so this module has a very large _fan-out_.  And, remember, there are five of these files.

Whenever a new device is added, all five files must be updated with the new device.  Whenever any of the `#include`d header files change, all five files must be recompiled.

Where are the device IDs like `CONSOLE` and `PAPER_TAPE_READER` defined?  They are `#define` macros in the `devids.h` file.  Whenever a new device is added to the OS, the `devids.h` file must be modified to add the new `#define` macro.  And, of course, any source file in the OS that `#include`s `devids.h` will have to be recompiled.

What this means for the plugin architechture is that you can't simply write the plugin.  You must instead modify, and increase the _fan-out_ of, a large and growing source module, and somehow link that in with your new plugin.  It's possible to do this with DLLs/Jars/SharedLibs, but it's not fun.

It also introduces an administration headache.  In a multiple team environment, you've got to keep control over those switch statements.  You don't want to create _DLL-HELL_ by allowing multiple teams to have their own versions.  

---

The vtable approach used by UNIX changes things around significantly.  The IO drivers can be loaded at any time.  When an IO device is selected the five functions are simply loaded into the `file`'s vtable.  There is no growing source file that steadily increases in _fan-out_.  There is no `devids.h` file to spur extra recompiles.  When new devices are added, nothing else has to be recompiled or relinked.  The DLL is simply loaded and registered into the set of devices.

---

Now the problems I've just outlined are quite common in modern applications.  It is not at all uncommon to see switch statement scattered throughout the body of the code -- all with the same cases but with different targets.  There can be a lot more than five; and they reproduce like gerbils.  

But this is likely a good place to pause and say that I think there is a time and place for both link-time and run-time binding.  There is a time and place for both if/switch and dynamic polymorphism dispatch.  

I think it is fair to say that I lean more towards the runtime polymorphism side of that divide.  But then I don't work in constrained environments.  Memory and cycles don't mean a lot to me anymore.  What matters most to me is source file organization and the minimization of the impact of source code dependencies.

---

Lastly, I just watched a video you did some time back on the principles of Reuse.  https://youtu.be/ZQ5_u8Lgvyk.  This is a great video.  It is densely packed with very good information that you presented very competently.  

In that video you told the story of how you tried, and failed, to create a reusable framework.  You said that failure, and the subsequent more successful attempts, caused you to learn a lot.  That learning was the source of the principles you presented.

Your experience is eerily similar to one I had in the early '90s.  My team and I had 36 applications to write in C++.  We had limited time.  The applications had many similarities and we pitched a reusable framework to our customer.  We had no idea how hard this was going to be.  Our first attempt at this framework was developed in concert with ONE of the 36 applications.  It tooks us a very long time to create.  When we were done we tried to reuse that framework in four more applications, and failed horribly.  So we adopted a new strategy.  We stripped that framework down and rebuilt it; but only with elements that were used in all four of the applications we were writing.  Again, it took a long time, but when we started the next four applications, the framework fit like a glove; and we finished all 36 well in advance of our deadline.

We learned a lot from that experience.  That learning is one of the sources of the SOLID principles, and the Clean Code strategies that I recommend to others. 

**CASEY**: I apologize for trying to be very specific here, but, I really want to actually get the exact proposal, and it wasn't really clear what it was. Could you tell me _exactly_ what the OS interface looks like and how it's implemented? You said _"I guess that depends a lot on the language and the application"_, but my understanding was that we were talking about the _OS_ side. Because obviously the application does not implement the raw device, so it doesn't need any switch statements or inheritance hierarchies or anything else. The OS and/or the driver implements the raw device, and it is expected to work across a wide range of applications and languages. So it can't "depend on the application or language" per se, it has to work with all of them. If you just mean it depends on what language the OS was written in, I assume it would be C/C++ because that's what Linux and Windows are written in at that layer, and I was hoping for this to be a real-world example.

In case it is not clear what I'm asking, we're talking about switch statements vs. inheritance hierarchies for implementing the layer from the OS down to the hardware. You said you thought this was a good example for where programmer cycles would be saved using inheritance vs. using switch statements. I assumed this meant that you believed having a particular inheritance hierarchy structure at the OS level would save either a) the OS programmers, b) the hardware vendors, or c) both, "programmer cycles", possibly at the expensve of CPU cycles, over the course of maintaining this OS. In my head, I don't see how that could possibly be the case vs. an enum/flags/switch/if approach. So I was hoping for an answer that spelled it out for me.

So what is the actual series of steps that happens here where the programmer cycles are saved? And I was hoping to then ask some hypotheticals, like about maintenance that would have to occur (adding a new device driver, etc.) Like, we start with the basic implemenation that they maybe had when they ship the first time, then we imagine the boss coming down and saying "now we need to support X", and then you can show me how it saves programmer cycles in that case, etc.

I'm assuming there is something like this somewhere?

```
class raw_device
{
public:
	virtual bool read(size_t Size, void *Memory) = 0;
	virtual bool write(size_t Size, void *Memory) = 0;
	// Is "close" here? "open"? A constructor or destructor? I'm not sure what you would advise.
};
```

Could you basically write what the base class is, and then how wiring works to go from the "getchar()"/"putchar()" example on the app side to the driver implementation of the derived class? That would be very helpful as a starting point so I can understand what you're actually recommending.

**Bob**: Woah, I had to update my RSA key for github.com before I could pull this.  See https://github.blog/2023-03-23-we-updated-our-rsa-ssh-host-key/

OK, so let's assume that we are writing a new OS.  And let's say that we are _not_ planning to hot-load the io drivers dynamically at runtime.  Instead we are going to statically link the OS.  Thus the only difference that we will consider between the switch and polymorphism solutions is programmer cycles. 

In my previous response I showed what I thought the `switch` statement implementation might look like.  
So here's what I think the dynamic polymorphism example might look like.

If we were writing this in C++ then there would likely be a base class defined as you showed above.
	
	#include "file.h"
	class raw_device {
	public: 
		virtual file* open(char* name) = 0; // some other args here too elided for simplicity.
		virtual void close(file* f) = 0;
		virtual void read(file* f, size_t n, char* buf) = 0;
		virtual void write(file* f, size_t n, char* buf) = 0;
		virtual void seek(file* f, int n) = 0;
		virtual char* get_name() = 0; // return the name of this device.
	}

>_Forgive my archaic C++ style, it's been 20+ years since I wrote any serious C++ or C._

In this example the `raw_device` is really nothing more than a jump table.  It holds no data or state.  The `file` data structure holds the state of the currently open session. The `raw_device.h` file has a source code dependency on `file.h`; and will never have another source code dependency on anything else.  It has a _fan-out_ of 1.  

Now I'd like to add an IO driver.  So I write the following code in `new_device.h`

	#include "raw_device.h"
	class new_device : public raw_device {
	public: 
		virtual file* open(char* name);
		virtual void close(file* f);
		virtual void read(file* f, size_t n, char* buf);
		virtual void write(file* f, size_t n, char* buf);
		virtual void seek(file* f, int n);
		virtual void get_name();
	}
	
I also write the implementation in a .cc file `new_device.cc`:

	#include "new_device.h"
	
	file* new_device::open(char* name) {...}
	void new_device::close(file* f) {...}
	void new_device::read(file* f, size_t n, char* buf) {...}
	void new_device::write(file* f, size_t n, char* buf) {...}
	void new_device::seek(file* f int n) {...}
	void new_device::get_name() {return "new_device";}
	
These two source files have a direct _fan-out_ of 1 and a transitive _fan-out_ of 2.  

Now we need a module to create the instances of our IO drivers and load them into a map of io devices keyed by their names.  My STL knowledge is a bit rusty so I'll make up a `map` class for this example.

	  io_driver_loader.cc
	  #include "new_device.h"
	  #include "new_device2.h"
	  #include "new_device3.h"
	  ...
	  
	  void load_devices(device_map& map) {
		  map.add(new new_device()); // map.add uses get_name to associate the device with the name.
		  map.add(new new_device2());
		  map.add(new new_device3());
		  ...
	  }
	  
>_I'm not compiling any of this so there are likely some syntax errors here and there that I hope you can forgive._

This last module has a _fan-out_ of N where N is the number of devices.  

Whenever a new IO device must be added the new .h and .cc file must be created.  And the `io_driver_loader.cc` file must be modified.  They'll all have to be recompiled and relinked.  

So let's count the programmer cycles in order to add a new device.

  * In the dynamic polymorpism case:
    * create `new_device_4.h`
    * create `new_device_4.cc`
    * modify `io_driver_loader.cc`
	* recompile and relink 3 files.

  * In the switch statement case:
    * create `new_device_4.h` forward declarations of the five functions.
	* create `new_device_4.cc` implementations of the five functions.
	* modify five switch statements, one for each of the functions, possibly in five different files.
	* modify `devids.h` to add the new `#define` macro for the device.
	* recompile and relink between 4 and 8 files, and all files that depend upon `devids.h`.

It should be clear that if we consider dynamic loading, then the `io_driver_loader.cc` file becomes something entirely different, and does not require modification when new IO devices are added; whereas the switch solution remains unchanged.

**CASEY**: Well, hold on, since I'm the switch statement proponent, I get to write the switch version :) But before we do that, I have a couple questions.

First, this is a raw device, so we don't actually need a file (I was trying to give us the simplest case so there would be as few details as possible!). So following your example I assume the base class in the DDK would be just:

```
	class raw_device {
	public: 
		virtual void read(size_t offset, size_t n, char* buf) = 0;
		virtual void write(size_t offset, size_t n, char* buf) = 0;
		virtual char* get_name() = 0; // return the name of this device.
	}
```

and the implementation in the vendor's driver would be

```
	#include "raw_device.h" // from the DDK
	class new_device : public raw_device {
	public: 
		virtual void read(size_t offset, size_t n, char* buf);
		virtual void write(size_t offset, size_t n, char* buf);
		virtual void get_name();
	}
```

Also, separate from the file handle part, I assume whenever one of these devices is used, it is looked up via the device map like this:

```
raw_device *find_raw_device(char *name) {
		raw_device *device = global_device_map[name];
		return device;
	  }
```

Is that correct? If you think it's important that the example include actual file handles, we can still do that, but if we do, I have other questions about the implementation.

**Bob**:  
 * Regarding the switch:  Sure, no problem.   
 * Regarding the file: My intent was that the file was an opaque pointer to _something_ that held the state of the session.  If you'd rather not have it, that's fine with me.  However, that reduces the number of operations from 5 down to 2 and impacts the count of programmer cycles.  But let's see where this goes.  

**CASEY**: We can definitely keep file in there if you'd like. It's your choice. I just figured I'm about to ask a bunch of questions about how you modify the design to accomodate OS and device upgrades, and it seemed easier if it was the minimal function set to start with (we can add more, like OSes have to do from time to time). But if you'd like to have file in there as well, we can add it back in now. I'd just have some questions about it if it's still going to be in there.

Assuming we _don't_ add it back in, my first question would be, what does the user-side version of this look like? In other words, I assume what you've designed here is in the OS and driver internals. What API does the application programmer use to interact with a raw_device?

**Bob**:
A lot depends in the kind of OS we are using.  We could be talking about a big fat memory mapped OS like MACOS, or a little embedded RTOS.  So my description below will try to cover the range without getting too tangled.

Leaving the `file` out of things for the time being, and presuming the `class raw_device` that you presented above lives on the OS side, then the lowest level user API would likely be a linkable library that provides access to primitive delegator functions: `read` and `write`.  These functions would likely have signatures like:

 * `read(char* name, size_t offset, size_t n, char* buf);`
 * `write(char* name, size_t offset, size_t n, char* buf);`
 
These functions delegate to implementations that exist in the OS.  These implemementations use the `find_raw_device` function that you suggested to delegate to the `read` and `write` functions of the selected instance of `raw_device`. 

Some means must be devised to implement the delegation across the OS boundary.  Some OSes might use special _interrupt_ or _system-call_ hardware instructions that the OS has provided vectors for.  Other OSes might create a vector table in some known part of memory.  Still others might simply declare the target functions to exist in a portion of memory that is memory mapped to a constant address.  

Another possibility would be to avoid the delegation altogether by making the entire IO subsystem of the OS a linkable library.  (We see this approach used in embedded OSes). In this case the low level APIs would use the `class raw_device` and the `find_raw_device` function without the imposition of the `read` and `write` delegator functions and implementations I described above.  
 
Above that primitive level of the API would be functions that are designed to be linked into the app, and that provide slightly higher level functionality.  For example, a function like `getchar` would be written in terms of calls to `read` (or in the embedded case direct calls to the `read` method of `raw_device`) and would use some means (such as environment variables) to translate `stdin` to the device name.

This is just one possible set of scenarios of course.  Others might involve exporting the vtables of the instances of `raw_device` for the current `stdin` and `stdout` devices so that they can be directly called by the application.  Indeed, this is close-ish to the approach used by some UNIX variants.

---

Having written all this, I fear we are getting fairly far afield from the actual topic, which is programmer-cycles vs. machine-cycles.  The example I presented was on the OS side only.  It seems to me that we'd be better off investigating whether that approach saves programmer cycles for the OS developers rather than worrying about the app developers on the other side of the OS boundary.  

Or perhaps we should focus on the embedded case where the OS and the app are linked together into a single executable.  In that case the OS developers and the app developers are harder to separate since the app developers will likely need to write their own device drivers.

---

OR -- perhaps we've gone down a rathole and it's time to cut to the chase.  See `programmer-cycles-vs-machine-cycles.md` file that I just created in this repository.  

As much as I've enjoyed this discussion, and as much as I've come to appreciate your knowledge and experience, I'm coming to the end of the time I can give to it.  So if our current thread is headed somewhere specific then let's get there without further delay. 

**CASEY**: I'm not even talking about machine-cycles, I was _just_ focusing on programmer-cycles. But I understand your design now, which I will summarize here for clarity.

First, every operation is represented by a function, 2+d times (once in userland, once in the kernel, then in d drivers). So for o operations, we have (2+d)\*o pieces of code, generally speaking.

On the application side we have:

```
int read(char* name, size_t offset, size_t n, char* buf);
int write(char* name, size_t offset, size_t n, char* buf);
```

These operate in userland and then they thunk down to ring-0 versions that look like (if I understand correctly):

```
int read_internal(char* name, size_t offset, size_t n, char* buf)
{
	// Translating addresses and checking bounds goes here, for "name" and "buf"

	int Error = DEVICE_NOT_FOUND;
	raw_device *Dev = find_device(name);
	if(Dev)
	{
		Dev->read(offset, n, buf);
	}
	
	return Error;
}
```

with raw_device being this:

```
	class raw_device {
	public: 
		virtual void read(size_t offset, size_t n, char* buf) = 0;
		virtual void write(size_t offset, size_t n, char* buf) = 0;
		virtual char* get_name() = 0; // return the name of this device.
	}
```

So, for comparison, let's look at what an enumerant-based solution would be. In this scenario, there are no practical limits I can think of in breaking things down into numbers, so I would do it for literally everything (the devices _and_ the operations):

```
// Some people prefer namespaces and such, so if that is your thing, imagine these are wrapped for naming however you prefer

enum raw_device_operation : u32
{
	RIO_none,
	
	RIO_read,
	RIO_write,
	RIO_get_name,
	
	RIO_private = 0x80000000, // Completely optional - I would do it, but then again, maybe I wouldn't if these drivers are ring-0 and we don't trust them much?
};

struct raw_device_id
{
	u32 ID;
};

struct raw_device_request
{
	size_t Offset;
	size_t Size;
	void *Buffer;
	raw_device_operation OP;
	raw_device_id Device;
	
	// Anything in an OS like this I typically want to pad to cache line size, in case there is multithreading going on:
	u64 Reserved64[4];
};

strcut raw_device_result
{
	u32 error_code;
	
	// Same as above - maybe this is overkill, but, I don't like cache-unaligned things, so, force of habit.
	u32 Reserved32;
	u64 Reserved64[7];
};
```

Now, I don't like vtables or language-specific object stuff pretty much anywhere, because I find it harder to control, so I would prefer

```
typedef void raw_device_handler(u32 Instance, raw_device_request *Packet, raw_device_result *Result);
struct raw_device
{
	raw_device_handler *Handler;
};
```

instead of the class version. But the only real reason I have to write that is, of course, because C++ is crappy and you can't say "I would like this class to contain function pointers, instead of a pointer to a table of function pointers, please." Otherwise, I wouldn't have a problem with the class version. So, although I would rather have the direct pointer version in practice, for purposes of this discussion (since we only care about programmer cycles right now), you can ignore my raw_device struct and we can just assume that it is written in class form like yours since I can't think of any programmer-cycles benefit to my version in this case:

```
class raw_device
{
public:
	void Handler(raw_device_request *Packet, raw_device_result *Result);
};
```

Since the user never sees a raw_device - they would just be using the 32-bit raw_device_id handle - leaving this as a class is fine.

When someone implements a device driver, they really just implement one function:

```
void raw_device::Handler(raw_device_request *Packet, raw_device_result *Result)
{
	switch(Packet->Op)
	{
		case RIO_read:
		// etc.
		
		case RIO_write:
		// etc.
		
		case RIO_get_name:
		// etc.
		
		default:
		// write error Result
	}
}
```

Obviously, the user-side version of this can be the same as yours:

```
int read(char* name, size_t offset, size_t n, char* buf);
int write(char* name, size_t offset, size_t n, char* buf);
```

However, I might recommend that it be more like:

```
raw_device_id lookup_device(char* name);
int read(raw_device_id device, size_t offset, size_t n, char* buf);
int write(raw_device_id device, size_t offset, size_t n, char* buf);
```

in _either_ case, but that is largely irrelevant, because that's solely about the lookup to map to a specific device, and can be used with either design.

Anyway, over the course of the development of the OS, I think this implementation _saves_ programmer cycles, potentially a lot of them, compared to the one I understand to be favored by the "Clean Code" method (which is also the one you wrote above). Again, in my understanding, the reason that yours looks the way it looks is specifically because of two design ideas that you mention often in books and in lectures:

* You favor having one class for each type of thing (in this case a driver), with one virtual member function per operation that the thing does (in this case read, write, and get_name)
* Operation permutations are not supposed to pass through functions - so passing an enum to a function that says what to do is bad, you should have one function per thing that gets done. I've seen this mentioned multiple times - not just for switch statements, but also when you say functions with "if" statements on function parameters in them, that modifies what the function does, you're supposed to rewrite that as two functions.

In my experience, these two things cost a lot of programmer cycles. I prefer to collapse things into functions with parameters when possible, because it reduces the total number of things that the programmer has to think about, and reduces the number of files, lines of code, and data permutations they have to consider. It allows functions to "pass through" information about what is happening without knowing what it is. Again, I tend to think of this as "operation-primal", because instead of focusing on making your _types_ polymorphic, you're making your _functions_ polymorphic.

I don't know if "abstracting file IO" is the best example to pick, but it was the first one you brought up, and it _happens_ to contrast the two designs enough in my opinion, so it works for me. Here's why I think an enum-based design saves the programmer(s) cycles:

* In most systems, we do not know all the functions that we are going to do ahead of time. When operating across a hard boundary like a driver, using operation codes instead of virtual function calls allows us to add functions dynamically without recompiling all of our drivers. If we want, for example, to add a new function like "trim" (such as what happened with real IO protocols when SSDs happened), if we are just passing opcodes to a driver, we don't have to recompile _anything_. All the old drivers just ignore the code, whereas the new drivers act on it, which is what we want. In the class case, either all the drivers have to be recompiled, or we have to write a utility class which stubs the new function, and wrap all the old drivers in that utility class. If we don't do this, the vtables for the old drivers will all be the wrong size, so we can't use them.
* In any modern system, multithreading is a concern, but this is especially true for an operating system. Having the protocol be structure-based, with an operation code, allows us to trivially buffer operations in things like io rings or other intermediaries _without writing any new code_. The entire system remains identical, and we need only add the new buffer logic. In the class system, we don't have any way to represent function operations with data, so wse must _introduce_ a data format for storing the calls - which of course will end up looking basically like the code for the non-class version, so you end up having to write my version _as well as_ the class version. This, by the way, seems to happen in almost _all_ OOP systems I see, because eventually they need to serialize or something similar, and so they have to write my version _as well as_ their version, but they don't seem to realize how much time they're wasting this way!
* If at some point we decide that users should be able to do multithreaded/bulk IO ops, with the enum version there are _zero_ changes to the internals and the drivers. All we do is add whatever user-land interface we want for this (presumably a circular buffer they write to), and everything "just works", with no translation necessary because the internals already worked on data instead of vtables/function calls anyway.
* If we would like to allow 3rd parties to have private communication channels for their devices, where user-level code can do IO ops that are only defined on those devices, there is no way to do this if we stick with the idea that each operation has to be represented by its own virtual function. It requires a complete side-channel implementation that has to be created specially and deployed by each vendor separately, because we have no idea how many functions each device will need or what they will be called. With the enum version, it's trivial. Just reserve half the opcode range for private stuff, and let anything in that range pass through to the device. Now the vendor ships an enum table to the end users, or an SDK with function calls that write those enums for them if you prefer, and that's it. Whether or not you'd actually _want_ to do this in an OS is questionable, because you're trusting the driver more here, but the point is the architecture _makes it trivial_. Whether or not you should allow it on security grounds is another question, and may depend on the type of OS.

Also, I should mention, that all of those things I just listed were _actual_ things that happened to IO subsystems in OS design. So none of those are hypothetical things that don't usually occur, or which are cherry-picked to be beneficial to enum-based design. Those are basically just _the exact things that have happened_ over the past ~20 years of OS IO subsystem evolution in a broad sense.

So what is really going on here? The problem, in my opinion, is that programs have paths between things. I have one place, like in the user code, where someone wants to do something. And I have another place, perhaps very far away like in a driver, where that thing has to happen. If we follow the style of classes for everything with small virtual functions that do one thing, we require that _the entire path_ between the two be _widened_ to include every possible operation the system needs to do. But why? Why are we duplicating these functions everywhere? Why not just make the path narrow, and use enums so that anyone who _wants_ to take action on the specific operation can, but those who don't want to don't have to?

Another way of saying this is that type-based polymorphism, despite its promises, actually _multiplies_ every path by the operation count, everywhere in the system, for no benefit. Instead of taking the entire path from the application to the device driver and making that _entire_ path "wide" in the sense that every function has an implementation at every stage, why not just use enums to collapse that pipeline into a single function the entire way, until you _actually_ need to fan it out?

Another way to say this is, if you can do multiple things with the same pattern, why not do them? What is the _benefit_ of multiplying the code?

It's worth mentioning at this point, as an aside, that I am not so much criticizing OOP in general here as I am its modern interpretation (as it is commonly practiced today and as it appears in Java, C++, and in "Clean Code" as well). I think the design I described with the enum would be _at least somewhat familiar_ to an early OOP programmer or advocate like Alan Kay, because it is _more like message passing_ which is what they were all about. And while I don't really think it benefits programmers to think in terms of collections of objects passing messages, I am less critical of _that_ idea than the virtual methods idea, and I do use things that look like message passing fairly often (as I did in this case with loadable drivers).

Now, in _most_ systems, I would _also_ be looking for ways to collapse across device types, too. But in this case, because we are specifically talking about a system where we say a priori that we want them to be loaded separately, then the only place we can do that is _inside_ the device driver, and we're not really talking about that part of the architecture. But if we were, I would be looking for those opportunities. So if, for example, several devices had similar interfaces, that only differed by a few things here and there, I would do _another_ non-"Clean Code" thing which is to collapse those into a single driver with "if's" based on the device types where appropriate.

I'll stop there, since I mentioned a lot of things, but hopefully that gives the general idea. I use this same approach basically everywhere - anything that looks similar gets collapsed into one thing, with an enum or a flags field that differentiates it. And it tends to produce the polar opposite of "Clean Code"-style code, because that style typically does the opposite: it creates the _maximum_ number of types and functions, whereas I am trying to produce a much smaller number - perhaps not the _minimum_, but certainly much less than Clean Code.

**Bob**:
OK, I think I see where you are going with this.  So first let me say: "Sure, looks good to me."  The bullet points you added after the fact are all quite valid and the design you picked works well in that case.  

In the first point you assume that operations will increase beyond the two originally proposed.  And as we both agreed before, and as I wrote in _Clean Code_ (which, by the way, is not the same as your "Clean Code") when operations proliferate more rapidly than types, switch statements are better.

In point two and three you raise the specter of muti-threading.  You are, of course, correct that queuing up operations is a lot easier if you use request packets of the type you designed.  No argument there.

The last point proposed a kind of "hook" for unknown and unspecified possibilities in the future.  OK, if you think those unknown and unspecified possibilities are likely then you have to consider them early; but that raises a number of other concerns that we should likely not address in this document.  So I'll let that point pass.

So now, where are we?

You posed a solution that uses dynamic polymorphism to select between device types, and then a switch statement to select operations.  I have no problem with this.  It works well, and satisfies my concerns about dependency inversion and programmer cycles.  I will say, however, that it is ironic that after your video, and after all the stress that has been put on saving machine cycles, you eventually chose a design that sacrifices machine cycles to save programmer cycles.   After all, on the OS side you've got to package up that request packet, hand it to the dynamically dispatched handler, and then run the operation ID through a switch.  ;-)

And so I think we wind up in the same place.  When operations proliferate more rapidly than types we both use switches.  When types proliferate more rapidly than operations we both use dynamic dispatch.  We are both willing to sacrifice machine cycles to save programmer cycles.

We are two individuals on the same island, the only difference being that I wear a shirt emblazoned with _Clean Code_, and you wear one emblazoned with "Clean Code".  

Thank you for a stimulating debate.  I appreciate the candor, and the civility that you exercized _here_, if not everywhere else.  I have come to respect your experience, your knowledge, and your articulation.  Writing is hard, and you appear to have mastered the art along with the art of software.  I hope in future we can do this again.  And if you ever want a recommendation to a publisher, let me know.

**CASEY**: Well, I disagree with most of that, but if we're ending it here, I'll just add my final responses for github posterity:

* Regarding _"as I wrote in _Clean Code_ (which, by the way, is not the same as your "Clean Code"),"_ well, the point of this discussion was for you to elaborate on what is not the same. But your design for this IO system looks exactly like my "Clean Code" example - a virtual function for every operation, and one class per element in the system with no predication. So what are these differences that you're referring to? Now would be the time to explain what they are, since that was the point of the concrete example. If this is a bad example for illustrating the differences, that's fine, but it was the first one you gave so I assumed it would be one you'd want to use.

* Regarding _"when operations proliferate more rapidly than types, switch statements are better,"_ that was not the case here. In no way are operations proliferating more rapidly than types in this system. Vendors will add drivers to an OS constantly, perhaps monthly or even weekly, whereas the number of operations in a particular system tends to go up much more slowly (once every few months at maximum, but more like once a year for something like an IO subsystem). It is the opposite of what you said. This is an important distinction, because what I am demonstrating here is the _opposite_ of your rule: this is showing that _even in the case where types proliferate far more rapidly than operations_, as is the case with drivers in an OS, the principle doesn't work. Enums are better _in both cases_. _Specifically because_ you have potentially thousands of types in the system (all the different drivers all the vendors have ever shipped), adding a single operation, however rarely, can cost _massive_ programmer cycles due to the unnecessary work multiplication across types that vtables cause. Another way to say this would be, enums are _more_ important in a system where types proliferate rapidly, _not less_.

* Regarding _"you eventually chose a design that sacrifices machine cycles to save programmer cycles,"_ I did no such thing. This design achieves both, that's why I like it. It is _dramatically_ faster to use something like a packet-based system than something like your original proposed design, because you do not take a ring transition on every operation. New OS IO APIs are now all designed this way: the user writes data without talking to the OS, and a kernel thread picks up those data writes. Nobody ever makes a function call, except occasionally to ensure the kernel thread hasn't gone to sleep :)  This is what I meant by the bullet point, "If at some point we decide that users should be able to do multithreaded/bulk IO ops" I am talking about the necessity that _actually occurred_ in both Linux and Windows of removing the frequency of ring transitions _for saving CPU cycles_. None of this is trading CPU cycles for programmer cycles. It's achieving both. The Linux kernel design of io_uring looks like my design! They did not add that to save programmer cycles. They added it because they wanted the highest possible IO throughput. This is an almost universal principle of modern OS design: anything that can be turned into data writes should be, and function calls should be minimized. It's been true for GPUs, for NICs, and for our example, disk IO.

* Regarding _"And so I think we wind up in the same place.  When operations proliferate more rapidly than types we both use switches.  When types proliferate more rapidly than operations we both use dynamic dispatch,"_ Again, I don't see how you got there. Obviously types are proliferating more rapidly in this system, so that part is not true. If we don't believe drivers are proliferating rapidly, why are we loading them dynamically? I thought that was the entire point of the example! But perhaps more importantly, we are not "using dynamic dispatch" here in the way you seem to be suggesting. As I said when I posted the proposed design, I would _also_ do this inside the drivers themselves. I _would not_ duplicate drivers to remove if statements and switches inside a driver that allowed that driver to handle multiple similar devices. The only reason there are function pointers in this system is because the problem definition required that we load the driver from a different module, and we are not presuming a JIT or something which can weld things together for us. That introduces a mandatory cut, so we cannot get rid of it because the problem is defined to contain it.

But note that that is _not_ the same between our two approaches. I have the function pointer there because it's required. And you'll note I minimized the number all the way down to one. I didn't put it in there because I think it saves programmer time. In fact, I'm not really sure I want it there at all. I haven't actually implemented this particular system in an OS, so it was somewhat off the top of my head, but _it's very possible_ that if I actually went to write this, I wouldn't actually include that function pointer at all. Instead, I might just have the OS thread reading the queue and prefiltering the packets for quota/permissions, then updating a shared memory address that lets the driver know it can process the packets directly. Without actually implementing, I can't say that's for sure what I would do, but, it's probably something I would try.

So it feels like you're overstating the similarity of our approaches, but, if you think they're that similar, then, I guess that's just where we end up! Thanks for taking the time to create this thread which pushed the github emoji checker well beyond its limits.

**Bob**: Thank you Casey.  I believe we should let our disagreement stand at this point and let our audience be the final judge.  This has been fun.  Perhaps we'll do it again some day.  

