---
layout: post
title:  "ChatGPT C# Code Review"
date:   2023-01-19 08:10:00 -0300
categories: programming AI ChatGPT C#
---

The Dutch government released some code as open source, and a particular
function generated lots of comments on Twitter:

>The Dutch government was forced to release the source code of their DigiD digital authentication iOS app. It is written in C#.

[https://twitter.com/JeroenFrijters/status/1615204074588180481](https://twitter.com/JeroenFrijters/status/1615204074588180481)


[https://github.com/MinBZK/woo-besluit-broncode-digid-app/blob/master/Source/DigiD.iOS/Services/NFCService.cs#L182](https://github.com/MinBZK/woo-besluit-broncode-digid-app/blob/master/Source/DigiD.iOS/Services/NFCService.cs#L182)


<p>I had an interesting conversation with ChatGPT about the code.</p>

I wonder what C# developers thing about ChatGPT's suggestions!
<br>
Please comment here: [https://twitter.com/tivrfoa/status/1616058016075243523](https://twitter.com/tivrfoa/status/1616058016075243523)

<div style="background-color: #a2b9bc; padding: 15px; border-radius: 10px; margin-bottom: 10px">
Me: Can you improve the code below?
</div>

```c#
        private static string GetPercentageRounds(double percentage)
        {
            if (percentage == 0)
                return "⚪⚪⚪⚪⚪⚪⚪⚪⚪⚪";
            if (percentage > 0.0 && percentage <= 0.1)
                return "🔵⚪⚪⚪⚪⚪⚪⚪⚪⚪";
            if (percentage > 0.1 && percentage <= 0.2)
                return "🔵🔵⚪⚪⚪⚪⚪⚪⚪⚪";
            if (percentage > 0.2 && percentage <= 0.3)
                return "🔵🔵🔵⚪⚪⚪⚪⚪⚪⚪";
            if (percentage > 0.3 && percentage <= 0.4)
                return "🔵🔵🔵🔵⚪⚪⚪⚪⚪⚪";
            if (percentage > 0.4 && percentage <= 0.5)
                return "🔵🔵🔵🔵🔵⚪⚪⚪⚪⚪";
            if (percentage > 0.5 && percentage <= 0.6)
                return "🔵🔵🔵🔵🔵🔵⚪⚪⚪⚪";
            if (percentage > 0.6 && percentage <= 0.7)
                return "🔵🔵🔵🔵🔵🔵🔵⚪⚪⚪";
            if (percentage > 0.7 && percentage <= 0.8)
                return "🔵🔵🔵🔵🔵🔵🔵🔵⚪⚪";
            if (percentage > 0.8 && percentage <= 0.9)
                return "🔵🔵🔵🔵🔵🔵🔵🔵🔵⚪";

            return "🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵";
        }
```

ChatGPT: Here's an improved version of the code:

```c#
private static string GetPercentageRounds(double percentage)
{
    int numRounds = (int)Math.Round(percentage * 10);
    return new string('🔵', numRounds) + new string('⚪', 10 - numRounds);
}
```

<p>This version uses the Math.Round method to round the percentage to the nearest whole number, and then uses that number to determine how many rounds of 🔵 to include in the returned string.</p>
<p>It also uses the string constructor that takes a character and a count to create a new string made up of that character repeated that many times. This eliminates the need for multiple if statements and makes the code more readable and maintainable.</p>

<div style="background-color: #a2b9bc; padding: 15px; border-radius: 10px; margin-bottom: 10px">
Me: Which of the two versions do you think is better?
</div>

ChatGPT: In my opinion, the second version of the code is better.
The first version uses multiple if statements to check a range of values and return a string based on that range. This can quickly become unwieldy if you need to add more ranges or if the ranges themselves are complex.<br>
<p>On the other hand, The second version uses the Math.Round method to round the percentage to the nearest whole number, and then uses that number to determine how many rounds of 🔵 to include in the returned string. This is more efficient and easy to read.</p>
<p>It also uses the string constructor that takes a character and a count to create a new string made up of that character repeated that many times. This eliminates the need for multiple if statements and makes the code more readable and maintainable.
So, the second version of the code is more efficient and readable.</p>

<div style="background-color: #a2b9bc; padding: 15px; border-radius: 10px; margin-bottom: 10px">
Me: Doesn't the second version creates new String objects on every call, hence causing an impact in GC?
</div>


ChatGPT: Yes, you are correct that the second version of the code creates new String objects on every call, which can have an impact on the garbage collector (GC) if called frequently. Creating new objects generates memory allocation and garbage collection can cause your application to pause.<br>
<p>The GC impact can be mitigated by reusing StringBuilder object that can be cleared and reused for every call,</p>
Here's an example of how it can be implemented:

```c#
private static StringBuilder stringBuilder = new StringBuilder(10);
private static string GetPercentageRounds(double percentage)
{
    int numRounds = (int)Math.Round(percentage * 10);
    stringBuilder.Clear();
    stringBuilder.Append('🔵', numRounds);
    stringBuilder.Append('⚪', 10 - numRounds);
    return stringBuilder.ToString();
}
```

<p>This version uses a static StringBuilder object that is created once at the start of the program and is reused for every call. The StringBuilder.Clear method is used to clear the contents of the StringBuilder object before it is reused, and the StringBuilder.Append method is used to append the appropriate number of 🔵 and ⚪ characters to the object. The ToString method is used to convert the StringBuilder object to a string, which is then returned. This eliminates the need for creating new string object for every call.</p>
<br>
Alternatively, you can use the string interpolation feature of C# to achieve the same result with less memory allocation, like this:

```c#
private static string GetPercentageRounds(double percentage)
{
    int numRounds = (int)Math.Round(percentage * 10);
    return $"{new string('🔵', numRounds)}{new string('⚪', 10 - numRounds)}";
}
```

It uses the string interpolation feature of C# to create the final string, without creating new string objects.


