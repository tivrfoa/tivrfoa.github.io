---
layout: post
title:  "James Gosling on NYJavaSIG 25th Anniversary"
date:   2020-09-25 9:26:44 -0300
categories: java software
---

[![James Gosling on NYJavaSIG 25th Java Anniversary](/assets/images/nyjavasig-25-anniversary.png)](https://www.youtube.com/watch?v=Yo-_x_px9h0)

Here I just marked some topics that James Gosling talked about.<br>
But you really should watch the whole video.<br>
Brian Goetz, Sharat Chander and Venkat Subramaniam are really great.<br>
Epic Event indeed!!!

[19:49](https://youtu.be/Yo-_x_px9h0?t=1189) - When did you start thinking of inventing Java?

[29:09](https://youtu.be/Yo-_x_px9h0?t=1749) - Why is it named Java?

[37:18](https://youtu.be/Yo-_x_px9h0?t=2238) - Original Java goal

[38:23](https://youtu.be/Yo-_x_px9h0?t=2303) - About C code
 
[38:43](https://youtu.be/Yo-_x_px9h0?t=2323) - About bytecode

[51:02](https://youtu.be/Yo-_x_px9h0?t=3062) - Brian Goetz asked if James wants to go back and work on Java with them.

[51:47](https://youtu.be/Yo-_x_px9h0?t=3107) - People on Amazon telling him about coding style and where he should place braces! Oh my gosh ...<br>
Check this [update](#update-2020-09-26) where James explains what he means by code density.

[1:02:03](https://youtu.be/Yo-_x_px9h0?t=3723) - Performance and value types

[1:04:15](https://youtu.be/Yo-_x_px9h0?t=3855) - Unsigned types and the unsigned right shift operator

[1:19:49](https://youtu.be/Yo-_x_px9h0?t=4789) - Don't do what people tell you to do, address their pain.

[1:31:12](https://youtu.be/Yo-_x_px9h0?t=5472) - Dafny programming language

[1:36:42](https://youtu.be/Yo-_x_px9h0?t=5802) - Is Java dead?

### Some quotes

>I would rather leave something out than put something wrong in.

>**Don't do what people tell you to do, address their pain instead.**
>
>In my career phase as a consultant I would never let people tell me
>what to do, or at least if they told me what to do I would ignore them.

About C:

>I had been writing C code for a long time and just how flaky, you know,
>it doesn't matter how talented you are at C code, it takes a long time
>to get stuff solid. And so that sort that sort of mindset drove things
>for me.

About bytecode:

>It was an actually a thesis proposal that I tried to get my advisor to
>accept when I was in grad school, but my advisors thought it was a bad
>idea.
>
>I had spent like five or six months on it and got shot down and did
>something else. But then doing this other thing I was like: oh that
>actually solves some real problems here. So I went back to it.


About Amazon telling James Gosling about coding style, and where he should place braces! lol

>At some point, you know when the project had got to about 15 or 20 people,
>they staged an intervention. You know it was kind of like you're
>an alcoholic and your family wants to inform you that you're an alcoholic.
>Because my coding style was really unusual. I favor density. I don't see
>code the way most people do. I don't read code top to bottom, I just like
>look at it and see it. You know, there's this sort of intervention over
>coding style because nobody was happy trying to keep up with my coding
>style.
>
>So, several runs through massive reformatters and they were all happy again.
>And a lot of it was like:
> - Where did the braces go?
> - How do you indent this?
> - What do variable declarations look like?
> - How many statements can you put on the line?
>
>There's a part of me that says you know, maybe James normal form is like
>the kernel of something else.


Then I had to search about James coding style, and I found a nice answer below:

[I've heard James Gosling has a very distinctive Java coding style. In what ways does it differ from more idiomatic Java?](https://www.quora.com/Ive-heard-James-Gosling-has-a-very-distinctive-Java-coding-style-In-what-ways-does-it-differ-from-more-idiomatic-Java)
> What I see is that sometimes he would omit braces from if and while loops, if the content of the block was only one line (sometimes even putting it on the same line). There is also sometimes the tendency to close a brace and start a new line for "else if", and also to omit blank lines between methods.

And one source code attributed to James. **I can't say if it really is**:
[BitReorder.java](https://gopherproxy.meulie.net/gopher.rbfh.de/0/Code/LinuxMagazin/2008/10/sprachen/BibReorder.java)

```java
/**
*
* @author James Gosling
*/
import java.io.*;
import java.util.*;
import java.util.ArrayList;
import java.util.regex.*;

public class BibReorder {
 String[] parts;
 DocumentFragment[] documentBody, bibliography;
 static Pattern bibRefPattern = Pattern.compile("\\[([0-9]+)\\]");
 public static void main(String[] args) {
   try {
     BibReorder doc = new BibReorder();
     boolean byFirstOccurrance = false;
     boolean processed = false;
     for(String s:args)
       if("-f".equals(s)) byFirstOccurrance = true;
       else {
         processed = true;
         doc.load(new FileInputStream(s));
         doc.reorder(byFirstOccurrance);
         doc.dumpDocument();
       }
     if(!processed) {
       doc.load(BibReorder.class.getResource("sample. Data").openStream());
       doc.reorder(byFirstOccurrance);
       doc.dumpDocument();
     }
   } catch (IOException ex) {
     System.err.println("Error reading file: "+ex);
   }
 }
 void load(InputStream in) throws IOException {
   Reader inc = new BufferedReader(new InputStreamReader(new BufferedInputStream(in)));
   StringBuilder sb = new StringBuilder();
   int c;
   while((c=inc.read())>=0) sb.append((char)c);
   parts = sb.toString().split("@footnote:");
   if(parts.length!=2) throw new IOException("Must have exactly 2 parts");
   documentBody = parseDocumentPart(parts[0]);
   bibliography = parseDocumentPart(parts[1]);
   in.close();
 }
 void reorder(boolean byFirstOccurrance) {
   int slot = 0;
   if(byFirstOccurrance) {
     for(DocumentFragment p:documentBody)
       if(p.tag!=null && p.tag.sequenceNumber==0) p.tag.sequenceNumber = ++slot;
     Arrays.sort(bibliography, new Comparator<DocumentFragment>(){
       public int compare(DocumentFragment o1, DocumentFragment o2) {
         return (o1.tag==null ? 0 : o1.tag.sequenceNumber)-(o2.tag==null ? 0 : o2.tag.sequenceNumber);
       }
     });
   } else
     for(DocumentFragment p:bibliography)
       if(p.tag!=null) p.tag.sequenceNumber = ++slot;
 }
 void dumpDocument() {
   dumpDocumentPart(documentBody, parts[0]);
   System.out.append("@footnote:");
   dumpDocumentPart(bibliography, parts[1]);
 }
 DocumentFragment[] parseDocumentPart(String s) {
   Matcher m = bibRefPattern.matcher(s);
   DocumentFragment prev = new DocumentFragment(0,null);
   ArrayList<DocumentFragment> pieces = new ArrayList<DocumentFragment>();
   pieces.add(prev);
   while(m.find()) {
     prev.end = m.start(0);
     String tlabel = s.substring(m.start(1),m.end(1));
     BibliographyTag t = tags.get(tlabel);
     if(t==null) { t = new BibliographyTag();
            tags.put(tlabel, t); }
     prev = new DocumentFragment(m.end(0),t);
     pieces.add(prev);
   }
   prev.end = s.length();
   return pieces.toArray(new DocumentFragment[pieces.size()]);
 }
 void dumpDocumentPart(DocumentFragment[] pieces, String part) {
   for(DocumentFragment p:pieces) {
     if(p.tag!=null) System.out.print("["+p.tag.sequenceNumber+"]");
     System.out.append(part,p.start,p.end);
   }
 }
 class DocumentFragment {
   int start, end;
   BibliographyTag tag;
   DocumentFragment(int st, BibliographyTag t) { start = st; tag = t; }
 }
 class BibliographyTag { int sequenceNumber; }
 HashMap<String,BibliographyTag> tags = new HashMap<String, BibliographyTag>();
}
```

### Update 2020-09-26

In the video below, James explains what he means with code density:

[James Gosling: Java, JVM, Emacs, and the Early Days of Computing | Lex Fridman Podcast #126](https://youtu.be/IT__Nrr3PNI?t=699)

>I find that I'm at odds with many of the people around me over issues about like
>how you lay out a piece of software. Software engineers get really cranky about
>how they format their programs you know, where they put new lines ... **And I tend
>to go for a style that's very dense, to maximize the ammount that I can see at once.**
>So I like to be able to see a whole function and to understand what it does, rather
>than have to go scroll scroll scroll and remember right.
>
>I'm sort of an odd person to be programming because I don't think very well
>verbally, I am just naturally a slow reader, **I'm what most people would call
>a visual thinker.** **When I look a program I see pictures.** It's almost like
>a piece of machinery with you this connnected to that ...
