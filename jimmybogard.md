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
