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