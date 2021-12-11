## LosTechies archive   
https://lostechies.com/jimmybogard/archive

## Best of
### 2007

#### https://lostechies.com/jimmybogard/2007/04/16/evils-of-duplication/

Monday, April 16, 2007
----------------------

### [Evils of duplication](https://lostechies.com/jimmybogard/2007/04/16/evils-of-duplication/)


Everyone is guilty of using "Ctrl-C, Ctrl-V" in code.  During development, we may see opportunities to duplicate code a dozen times a day.  We're working on some class Foo that needs some behavior that is already implemented in class Bar.  The quickest way to implement this is Copy, Paste, and call it a day.  But what are the long-term implications of this?  If this is a codebase that must be maintained for months, even years, what will these seemingly innocuous keyboard coding shortcuts do to us?  The answer is not pretty.

### A complex world

The truth is, coding isn't that hard.  Give me a feature, and on a clean slate system, I could have it done in two weeks.  I code it as fast as I can, and with some solid testing, I can be reasonably sure that you'll get what you asked for.

Fast forward six months.  Everything is great with the feature I implemented, but I'm requested to make some small change.  Now I'm in trouble.  The small change isn't that difficult, but now I need to figure out how that affects the existing system.  I'm in luck though, it looks like there's already something similar in the codebase, and a quick Copy-Paste second later, the changes are done (with some minor tweaks, of course).  I get tired of making these changes for these ungrateful customers, so I decide to leave for greener pastures and cleaner slates.

Now the customer wants another change, but this time it affects **both** features.  I'm not around, and I'm not answering emails, so it's up to Joe to fix this problem.  Not being familiar with the codebase, Joe resorts to searching for changes he must make.  He finds the two features (a couple of days later) and makes the change **in both places**.  I think you can see where this is going.  Any change that affects the two features will have to be made in both places, and it will be up to the developer to remember or be lucky enough to find the two places where the changes are made.  Instead of taking a couple of days to make a change, it takes a month, because Joe has to make darn sure that this code wasn't copied and pasted a dozen other times.

### Conditions deteriorate

If Copy Paste is rampant in a codebase, maintainability vanishes.  We'd spend 95% of our time just figuring out what the code is doing, where the changes should be made, and how many other places I need to fix since the logic is duplicated many times rather than actually coding the changes.  All because I decided to take a shortcut and Copy Paste.  Our estimates need to go up even though the changes aren't hard because we have no idea how many places we have to make the same fix.

### Real-world example

I had a defect recently where I needed to know whether the current web customer was a Business (rather than Individual) customer or not.  I found some logic in a base page class that looked like this:

```csharp
protected bool IsBusiness  
{  
   get  
   {  
      if ((StoreContext.Current != null) && (StoreContext.Current.StoreSettings != null) && (StoreContext.Current.StoreSettings.CustomerType != null) && (StoreContext.Current.StoreSettings.CustomerType.Trim().Length>0))  
      {  
         if (StoreContext.Current.StoreSettings.CustomerType.ToUpper() == "BUSINESS")  
         {  
            return true;  
         }  
      }  
      return false;  
   }  
}  
```

This looks pretty good, but it's on a Page class and I'm working with a UserControl.  I _could_ just copy paste this code block and be finished.  But what if this logic ever changed?  What if instead of "BUSINESS", the compare string needed to change to "CORPORATE".  As a quick sanity check, I looked for other instances of this logic to see if I couldn't use a different class.  Bad news.  This same block of code is used over a dozen times in my solution.  That means that this string can never change because the cost of change is too great and it affects too many modules.

### A solution

What I'd really like is for this logic to be in only one place.  That is, logic should appear [once and only once](http://c2.com/xp/OnceAndOnlyOnce.html).  Here's a static helper class I would use to solve this problem (with some minor refactorings\*):

```csharp
public static class SettingsHelper  
{  
  
   public static bool IsBusiness()  
   {  
      StoreContext context = StoreContext.Current;  
  
      if ((context == null) ||   
         (context.StoreSettings == null) ||   
         (context.StoreSettings.CustomerType == null))  
         return false;  
           
      string customerType = context.StoreSettings.CustomerType.Trim();  
  
      return (string.Compare(customerType, "BUSINESS", StringComparison.OrdinalIgnoreCase) == 0);  
   }  
}  
```

When I eliminate duplication, I increase maintainability and lower the total cost of ownership.  Duplication occurs in many forms, but Copy Paste is the easiest for the developer to eliminate.  Just don't do it.  If you feel the urge to use that "Ctrl-C, Ctrl-V" combination, stop and think of Joe, who has to clean up your mess.  Think about where this behavior should be, and put it there instead.  I'd try to put the behavior as close to the data as possible, on the class that contains the data if possible.  If it requires a helper class, create one.  Just do whatever it takes to prevent Copy Paste duplication, since we pay many times over in maintenance costs for maintaining duplicated logic.

### Greener pastures

We have a codebase that's been successful for years.  I'll try to treat it like my home, and clean as I go.  I won't introduce duplication, and I'll do my best to eliminate it where I find it.  Doing so will ensure that this codebase will continue to be successful in the years to come.  I've been that developer Joe and it's not fun.

\* Refactorings:

*   [Replace nested conditional with guard clauses](http://www.refactoring.com/catalog/replaceNestedConditionalWithGuardClauses.html)
*   [Introduce explaining variable](http://www.refactoring.com/catalog/introduceExplainingVariable.html)
*   [Replace ToUpper with string.Compare](http://www.codeproject.com/csharp/stringperf.asp)

Posted by Jimmy Bogard at [8:53 AM](https://grabbagoft.blogspot.com/2007/06/evils-of-duplication.html "permanent link")  [![](https://resources.blogblog.com/img/icon18_edit_allbkg.gif)](https://www.blogger.com/post-edit.g?blogID=3208248179046825763&postID=4395920468844431094&from=pencil "Edit Post") 


http://grabbagoft.blogspot.com/2007/06/feedback.html  
http://grabbagoft.blogspot.com/2007/06/notes-on-defects.html ([Correct link for Software Defect Reduction Top 10 List ](https://www.cs.umd.edu/projects/SoftEng/ESEG/papers/82.78.pdf]))  
http://grabbagoft.blogspot.com/2007/06/re-throwing-exceptions.html  
http://grabbagoft.blogspot.com/2007/06/swallowing-exceptions-is-hazardous-to.html  
http://grabbagoft.blogspot.com/2007/06/proper-string-comparison.html  
http://grabbagoft.blogspot.com/2007/06/pop-quiz-on-ref-and-out-parameters-in-c.html  

[CQRS/MediatR implementation patterns](https://lostechies.com/jimmybogard/2016/10/27/cqrsmediatr-implementation-patterns/)
====================================

27 October, 2016. It was a Thursday.[](#disqus_thread)

* * *

Early on in the CQRS/ES days, I saw a lot of questions on modeling problems with event sourcing. Specifically, trying to fit every square modeling problem into the round hole of event sourcing. This isn’t anything against event sourcing, but more that I see teams try to apply a single modeling and usage strategy across the board for their entire application.

Usually, these questions were answered a little derisively  – “you shouldn’t use event sourcing if your app is a simple CRUD app”. But that belied the truth – no app I’ve worked with is JUST a DDD app, or JUST a CRUD app, or JUST an event sourcing app. There are pockets of complexity with varying degrees along varying axes. Some areas have query complexity, some have modeling complexity, some have data complexity, some have behavior complexity and so on. We try to choose a single modeling strategy for the entire application, and it doesn’t work. When teams realize this, I typically see people break things out in to bounded contexts or microservices:

![image](https://user-images.githubusercontent.com/350314/145684051-dcec7fd4-a399-4b7b-a1c9-a8f65e1546b0.png)

With this approach, you break your system into individual bounded contexts or microservices, based on the need to choose a single modeling strategy for the entire context/app.

This is completely unnecessary, and counter-productive!

A major aspect of CQRS and MediatR is modeling your application into a series of requests and responses. Commands and queries make up the requests, and results and data are the responses. Just to review, MediatR provides a single interface to send requests to, and routes those requests to in-process handlers. It removes the need for a myriad of service/repository objects for single-purpose request handlers (F# people model these just as functions).

###

### Breaking down our handlers

Usage of MediatR with CQRS is straightforward. You build distinct request classes for every request in your system (these are almost always mapped to user actions), and build a distinct handler for each:

![image](https://user-images.githubusercontent.com/350314/145684057-01c441a2-7b06-41b9-ad13-0cfc87b85e88.png)

Each request and response is distinct, and I generally discourage reuse since my requests route to front-end activities. If the front-end activities are reused (i.e. an approve button on the order details and the orders list), then I can reuse the requests. Otherwise, I don’t reuse.

Since I’ve built isolation between individual requests and responses, I can choose different patterns based on each request:

![image](https://user-images.githubusercontent.com/350314/145684066-bb4b432c-776c-4586-94c2-a8bb8dba8b48.png)

Each request handler can determine the appropriate strategy based on \*that request\*, isolated from decisions in other handlers. I avoid abstractions that stretch across layers, like repositories and services, as these tend to lock me in to a single strategy for the entire application.

In a single application, your handlers can execute against:

*   [Transaction scripts](http://martinfowler.com/eaaCatalog/transactionScript.html)
*   [Domain models](http://martinfowler.com/eaaCatalog/domainModel.html)
*   [Service layers](http://martinfowler.com/eaaCatalog/serviceLayer.html)
*   [Event-sourced aggregates](http://martinfowler.com/eaaDev/EventSourcing.html)

It’s entirely up to you! From the application’s view, everything is still modeled in terms of requests and responses:

![image](https://user-images.githubusercontent.com/350314/145684076-154a1d2e-d26b-450d-b75a-f27d1b6162b7.png)

The application simply doesn’t care about the implementation details of a handler – nor the modeling that went into whatever generated the response. It only cares about the shape of the request and the shape (and implications and guarantees of behavior) of the response.

Now obviously there is some understanding of the behavior of the handler – we expect the side effects of the handler based on the direct or indirect outputs to function correctly. But how they got there is immaterial. It’s how we get to a design that truly focuses on behaviors and not implementation details. Our final picture looks a bit more reasonable:

![image](https://user-images.githubusercontent.com/350314/145684097-cefd9e11-543a-4766-8ca6-a4a1bfc85262.png)

Instead of forcing ourselves to rely on a single pattern across the entire application, we choose the right approach for the context.

### Keeping it honest

One last note – it’s easy in this sort of system to devolve into ugly handlers:

![image](https://user-images.githubusercontent.com/350314/145684112-47f13737-3f44-40a6-a239-b5729510848a.png)

Driving all our requests through a single mediator pinch point doesn’t mean we absolve ourselves of the responsibility of thinking about our modeling approach. We shouldn’t just pick transaction script for every handler just because it’s easy. We still need that “Refactor” step in TDD, so it’s important to think about our model **before** we write our handler and pay close attention to code smells **after** we write it.

Listen to the code in the handler – if you’ve chosen a bad approach, refactor! You’ve got a test that verifies the behavior from the outermost shell – request in, response out, so you have a implementation-agnostic test providing a safety net for refactoring. If there’s too much going on in the handler, push it down into the domain. If it’s better served with a different model altogether, refactor that direction. If the query is gnarly and would better suffice in SQL, rewrite it!

Like any architecture, one built on CQRS and MediatR can be easy to abuse. No architecture prevents bad design. We’ll never escape the need for pull requests and peer reviews and just standard refactoring techniques to improve our designs.

With CQRS and MediatR, the handler isolation supplies the enablement we need to change direction as needed based on each individual context and situation.

[Dealing with Duplication in MediatR Handlers](https://lostechies.com/jimmybogard/2016/12/12/dealing-with-duplication-in-mediatr-handlers/)
============================================

12 December, 2016. It was a Monday.[](#disqus_thread)

* * *

We’ve been using MediatR (or some manifestation of it) for a number of years now, and one issue that comes up frequently is “how do I deal with duplication”. In a traditional DDD n-tier architecture, you had:

*   Controller
*   Service
*   Repository
*   Domain

It was rather easy to share logic in a service class for business logic, or a repository for data logic (queries, etc.) When it comes to building apps using CQRS and MediatR, we remove these layer types (Service and Repository) in favor of request/response pairs that line up 1-to-1 with distinct external requests. It’s a variation of the [Ports and Adapters](http://alistair.cockburn.us/Hexagonal+architecture) pattern from Hexagonal Architecture.

Recently, going through an exercise with a client where we collapsed a large project structure and replaced the layers with commands, queries, and MediatR handlers brought this issue to the forefront. Our approaches for tackling this duplication will highly depend on what the handler is actually doing. As we saw in the previous post on [CQRS/MediatR implementation patterns](https://lostechies.com/jimmybogard/2016/10/27/cqrsmediatr-implementation-patterns/), our handlers can do whatever we like. Stored procedures, event sourcing, anything. Typically my handlers fall in the “procedural C# code” category. I have domain entities, but my handler is just dumb procedural logic.

### Starting simple

Regardless of my refactoring approach, I ALWAYS start with the simplest handler that could possibly work. This is the “green” step in TDD’s “Red Green Refactor” step. Create a handler test, get the test to pass in the simplest means possible. This means the pattern I choose is a [Transaction Script](http://martinfowler.com/eaaCatalog/transactionScript.html). Procedural code, the simplest thing possible.

Once I have my handler written and my test passes, then the real fun begins, the Refactor step!

**WARNING: Do not skip the refactoring step**

At this point, I start with just my handler and the code smells it exhibits. [Code smells](https://martinfowler.com/bliki/CodeSmell.html) as a reminder are indication that the code COULD exhibit a problem and MIGHT need refactoring, but is worth a decision to refactor (or not). Typically, I won’t hit duplication code smells at this point, it’ll be just standard code smells like:

*   Large Class
*   Long Method

Those are pretty straightforward refactorings, you can use:

*   Extract Class
*   Extract Subclass
*   Extract Interface
*   Extract Method
*   Replace Method with Method Object
*   Compose Method

I generally start with these to make my handler make more sense, easier to understand and the like. Past that, I start looking at more behavioral smells:

*   Combinatorial Explosion
*   Conditional Complexity
*   Feature Envy
*   Inappropriate Intimacy
*   and finally, Duplicated Code

Because I’m freed of any sort of layer objects, I can choose whatever refactoring makes most sense.

### Dealing with Duplication

If I’m in a DDD state of mind, my refactorings in my handlers tend to be as I would have done for years, as I laid out in my (still relevant) blog post on [strengthening your domain](https://lostechies.com/jimmybogard/2010/02/04/strengthening-your-domain-a-primer/). But that doesn’t really address duplication.

In my handlers, duplication tends to come in a couple of flavors:

*   Behavioral duplication
*   Data access duplication

Basically, the code duplicated either accesses a DbContext or other ORM thing, or it doesn’t. One approach I’ve seen for either duplication is to have common query/command handlers, so that my handler calls MediatR or some other handler.

I’m not a fan of this approach – it gets quite confusing. Instead, I want MediatR to serve as the outermost window into the actual domain-specific behavior in my application:

![image](https://user-images.githubusercontent.com/350314/145682813-63fe0265-e0d1-4593-b77e-95c4dc4876c8.png)


Excluding sub-handlers or delegating handlers, where should my logic go? Several options are now available to me:

*   Its own class (named appropriately)
*   Domain service (as was its original purpose in the DDD book)
*   Base handler class
*   Extension method
*   Method on my DbContext
*   Method on my aggregate root/entity

As to which one is most appropriate, it naturally depends on what the duplicated code is actually doing. Common query? Method on the DbContext or an extension method to IQueryable or DbSet. Domain behavior? Method on your domain model or perhaps a domain service. There’s a lot of options here, it really just depends on what’s duplicated and where those duplications lie. If the duplication is within a feature folder, a base handler class for that feature folder would be a good idea.

In the end, I don’t really prefer any approach to the another. There are tradeoffs with any approach, and I try as much as possible to let the nature of the duplication to guide me to the correct solution.
