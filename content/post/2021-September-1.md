---
date: "2021-09-03"
tags: ["AppSec", "Code"]
title: "Subtle Bugs"
---

Recently, I've been performing Manual Secure Code Reviews. The OWASP Top10 appear consistently but subtler bugs pop up too. I wanted to cover some of those in this post.

Control flow refers to the order in which statements are executed. Programs usually execute linearly. Functions, loops, and conditionals will modify that flow though. Conditionals create alternative pathways, loops cause repetition, and functions create entirely new execution paths on the control graph. 

# Bugs in Conditionals

Conditional statements are the most common modifier to control flow. These allow programs to act differently based on whether a condition is satisfied. `if` and `switch` are the most common conditional statements. Programmers have good intentions, but their assumptions about control flow (or just programming in general) can lead to security bugs.

```js
if (Age > 50) {
  print("Ok Boomer");
} else if (Age > 10) {
  print("Ok Zoomer");
} else {
  print("Ok Doomer");
}

switch (Pokemon) {
  case 'Vanilluxe':
    console.log('This Pokemon is not yet extinct sadly.');
    break;
  case 'Omanyte':
    console.log('This Pokemon is extinct (kind of).');
    break;
}
```

Conditional Statements tend to follow the form of `If`, `If-Else`, and `If-ElseIf-Else`. Forgetting to place your flow within an else statement can lead to unintended consequences. In the following scenario, regardless of the if statement, the jedi order is wiped out.

```js
if !authorized {
    doScream("I AM THE SENATE");
}
ExecuteOrder66();
```
A terrible fate to befall the galaxy, using an `Else` statement here may have prevented this tragedy. That said, using `Else` isn't always the answer. Here, the programmer here has made an assumption that cars stop on red lights only. 

```js
if (light === "red") {
    stop();
} else {
    go();
}
```
Emergency vehicles, animals, accidents, and pedestrians all break that assumption. Having catch-all statements can bite you. Spend time frontloading error scenarios instead of relying on `Else` or `Catch` to save your program. The above example might seem contrived, but there have been [recent cases](https://blog.truesec.com/2021/07/06/kaseya-vsa-zero-day-exploit/) where an `Else` statement was exploited. The below code shows a situation where an AuthN flow allows access if none of the `If` Conditions are true.

```js
if password == hash(row[nextAgentPassword] + row[agentGuidStr]) 
    login ok 
elseif password == hash(row[curAgentPassword] + row[agentGuidStr])
    login ok
elseif password == hash(row[nextAgentPassword] + row[displayName])
    login ok
elseif password == hash(row[curAgentPassword] + row[displayName])
    login ok
elseif password == row[password]
    login failed 
else 
    login ok
```
Moving on to short-circuit execution. Short-circuiting improves performance by stopping conditional evaluation as soon as the result has been determined. This is a blessing and a curse, short circuiting conditionals can lead to weird control-flow behaviour. 
```js
var state === null
if (state != null & state.toUpperCase() === "Disarray") // Crashes 
if (state != null && state.toUpperCase() === "Confusion") // Does not.
```
By using a non-short-circuiting operator here, the first statement will crash as it tries to dereference a null variable. The second statement will short-circuit and return False prior to the crash. In general, you should use Short-Circuit operators. But the order of your conditionals matters if you do.

```js
if (validToken() || validateInput()) // Abusable for skipping input Val
```
In this case, providing a valid token skips input validation. The main thing is to make sure the highest priority function is placed to the left as execution usually occurs left to right. Alternatively, structure your program to nest conditional statements to avoid this problem entirely.

A few more areas where bugs surface include: using the NOT (!) operand, brackets in logic, and using assignment in a conditional.

```js
if (!isExplicit) {
    HotTubStream(); // Not isExplicit is confusing.
}

if (young && wonderful || (beautiful || magical)){
    beResponsiblePractical(); // How does this evaluate?
}

if (search = document.getElementById("book")) {
    ShelveIt(); // Did you notice that an assignment was used for this conditional?
}
```
For all of these, write clear code. Don't use negation if you don't have to, simplify your conditionals, and don't use assignment within a conditional. The last one really stinks of javascript fuckery.

Arithmetic operations and truthiness also pop up occasionally when I review code.

```js
var place
if (place === 1 || 2 || 3) {
    AwardMedal(); // Medals for Everyone!!! - Soldier TF2
}

if (place >= 1 ) {
    AwardMedal(); // Medals for Everyone Positive!
}

if (place === 1 || place === 2 || place === 3) { 
    AwardMedal(); // Fixed
}
```
The first statement is a common junior programmer mistake. There are three conditions being tested: 
* `place === 1`
* `2`
* `3`

Regardless of the value of `place`, 2 and 3 are Truthy in JavaScript. This means that they are true when evaluated as a boolean. So the first statement will always evaluate to true. Medals for everyone!

![Medals for Everyone!](../../img/posts/2021-09/medals.png)

In the second statement, the programmer tried to give medals to 1, 2, and 3, but also every other positive placing. These arithmetic issues can be fixed by being explicit with each boolean comparison. Truthiness in Javascript can lead to some real dumb control flow situations, as can arithmetic assumptions.

```js
if (domain.includes("localhost") || domain.includes("127.0.0.1")) // Abusable
if (domain.host === "localhost" || domain.host === "127.0.0.1") // Fixed
```
Here, an adversary can use a domain like bigsexylocalhost.com to go down this control path, despite the best intentions of the programmer. This comes up quite often with regards to domains, IP addresses, WAF Rules, and file upload functionality. To fix this, be explicit and canonicalise input. Switch statements also fail for most of these as well, but you must consider judicious `break;` placement and `default:` in addition.

Finally, as far as conditionals go, a common area to forget about is TryCatch. Intentionally causing errors at certain points of an application can influence a program more than you would think.

# Bugs in Loops

Loops allow a program to repeat certain tasks until a condition (!) is satisfied. Sometimes the loop is meant to run indefinitely `while true` and in other cases it's meant to run a predefined amount `for each x in y`. 

Sometimes the bug is an input validation one as simple as allowing a user to control how long something executes.

```js
request.getParameter(number);
result = 1;
for (int i=1; i<=number; i++)
{
    result *= i; // Factorial
}
return result
```
Obviously, big numbers mean long execution time.

In a different vein, computational complexity can lead to a DOS as well. 
```js
for (int i=0; i<=countries; i++){
    for (int j=0; j<=states; j++){
        for (int k=0; k<=cities; k++){
            for (int l=0; l<=streets; l++){
                if locationMatch(i,j,k,l) {
                    return true
                }
            }
        }
    }
}
return false
```
While this scenario might not be too bad, after all, there is a pretty limited selection of countries and states, it could easily become a CPU hogging monster in other circumstances. 

# Final Thoughts

The higher the [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) of an application, the more likely you'll find subtler bugs like the ones mentioned here. Structuring your program to minimise control flow statements, writing clear code, and trying to be very explicit where possible are all ways you can help improve the security and reliability of applications you develop going forward.  and hey, if you're interested in getting me to take a look at your code sometime, let me know on LinkedIn. :D

Hope you enjoyed this article!