**Bob**:

Let's go right back to the beginning and talk about your [_"Clean" Code, Horrible Performance_](https://youtu.be/tD5NrevFtbU) video.

In that video you attacked my ideas rather ruthlessly from a single point of view -- performance.  That's fair.  I never suggested that performance was my main goal.  I want the act of programming to be easier, more pleasant, and more productive for the vast majority of programmers; and I am willing to sacrifice performance in order to achieve that.

Towards that end I have suggested, and will continue to suggest, the following things:

 * Carefully choose nice names.
 * Keep your functions small.
 * Keep your classes small.
 * Manage your dependencies.
 * Be careful with side effects.
 * Express yourself in code where possible.
 * Use polymorphism when types change faster than operations.
 * Use switch when operations change faster than types.
 * When possible create designs where the things that change fast are types.
 * Test Driven Development
 * ...and much, much, more.
 
You and I now appear to be chasing each other in a downward spiral towards a stalemate in the basement of some operating system somewhere.  We seem destined to lock horns over whether switch statements or polymorphism are best in all cases.  Let's not go there. 

The bottom line is this:

Switch statements have their place.  Dynamic polymorphism has its place.  Dynamic things are more flexible than static things, so when you want that flexibility, and you can afford it, go dynamic.  If you can't afford it, stay static.
