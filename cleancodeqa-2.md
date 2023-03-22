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

