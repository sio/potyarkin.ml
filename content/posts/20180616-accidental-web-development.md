title: Accidental submersion into web development
tags: python, web, HomeLibraryCatalog
date: 2018-06-16

## The Library Problem

I love reading books. My wife loves reading books. We enjoy shopping for books
and we live a ten minute commute away from a huge used books store. That means
we have a lot of books. Like, really a lot. A little more than one thousand.

We have lost count how many times we've bought a book that we already owned.
Even more often we had foregone buying a book we liked because we were sure we
have already bought it - only to find out we've been mistaken and have only
considered buying that very book earlier. This has become a problem. The Library
Problem.

We needed a way to catalog all our books. The catalog had to be accessible from
mobile devices (to look up a book while at the book store) and to be easy to
use. That is to add and edit book information, of which we've needed plenty: in
addition to standard set of author, title and publishing year we wanted to be
able to track book series and keep the list of missing books to look out for the
next time.

I admit that my research into the subject matter was not scientifically thorough.
I've dug up several comparisons of existing tools and have read several blog
posts of people who have faced the same problem before. I particularly recommend
[this one][blog]. And I have decided that no pre-existing tool will meet our
growing expectations.

[TOC]

## Naive foray into application architecture

I have never developed an application for any end user other than myself and I
didn't even know I was about to start developing one.

### Single spreadsheet approach

The initial idea was that of a "document". It had to contain some essential
information about every book we own and (optionally) a second list of books we
want to acquire. Excel spreadsheet seemed like a natural fit. We are both
spending a better half of each workday juggling spreadsheets in Excel, so
developing and maintaining such "document" appeared doable.

I have started drafting the column structure, applied some formatting tweaks and
the skeleton of the "document" was ready. We have entered the test batch of
books and were about to begin testing.

Obstacles arose when we were entering information for the first fifteen books.
Manually typing in all the fields is tiresome and slow. Errors inevitably
happen. What if I miss one letter in the author's name?  That book will be lost
when applying filter on author column. We need autocompletion! What if I
accidentally switch the order of digits in ISBN? We won't be able to do a web
lookup for that book later. We have to write a custom ISBN validator in VBA!

And the spreadsheet began to amass VBA code, data validation and conditional
formatting rules. Full-blown spaghetti style. Version control has become a
problem. What's the difference between versions 0.0.13 and 0.0.19? I had only a
vague idea.

I stopped myself when I was about to sketch up a UserForm for data input. Excel
road was leading me nowhere. It was difficult and even if all the difficulties
were to be overcome it imposed some suboptimal compromises on us:

- Single table storage limited the data structures we would be able to enter and
  view. If the book was written by two authors, which one should come first in
  the "author" string?  How would we do column filtering in that case? What
  about sort order?
- The local nature of storage meant there had to be one designated place for
  making changes (home laptop). Any changes made in other locations (smartphone,
  thumb drive) had to be agreed to be declared discardable.
- The spreadsheet had to be exported to HTML to be accessible from smartphones.
  XML and XSLT made this possible but not very pleasant. Although, I am rather
  proud of the VBA code I wrote to save/load XML data automatically upon opening
  the workbook. The data was completely decoupled from the representation.

I'm glad I did not waste more time pursuing this path, but it was still hard to
let go. It took quite some time for me to return to this project afterwards.

### Local database driven application with web interface

A relational database was the logical solution to spreadsheet limitations.
Store authors separate from books and manage how the former *relate* to the
latter! Scientists who pioneered the [relational model] in 1970s were pure
geniuses and now the whole world relies on their work.

I drafted the database schema on a piece of paper and have discussed it
extensively with my wife. That's probably the point where book reviews were
added to the requirements list. The database idea made me very enthusiastic and
had swallowed a lot of free time, but that idea alone could not provide a
complete solution for the problem at hand.

Data input and representation are what the Library project was all about. Yet
relational database management systems provide only the storage solution, not
the full package (if you don't take Microsoft Access into account). So I had to
figure out how to implement the user interface on my own. I chose to write it in
Python because I was somewhat familiar with the language and I enjoy how clean
and readable Python code is. I'm not a programmer, so other options were either
learning a new language or choosing between VBA and Bash, none of which could be
considered enjoyable.

I have briefly considered building a conventional desktop GUI. The UserForm
experience was still fresh and rather traumatic, so I was reluctant to dive into
another UI toolkit. Even if I were to, which one should I have chosen? Qt seemed
nice, but its Python support reports were contradictory. GTK? Installing it was
rather tricky in Windows. Something non-crossplatform? And what about my Linux
laptop?

I've had a bunch of HTML templates left from XML/XSLT operations and they could
be trivially transformed to be used in conjunction with the database. While the
catalog pages could be statically generated, the data input required some sort
of server to interact with. And I've had zero experience with that.

Quick Google search has introduced me to the concept of web frameworks and I
have semi-randomly chosen [Bottle]. At that moment I've had no intention to
expose it to the open network, the app and the database were to be stored on the
USB stick and launched locally when needed. Smartphone interaction was planned
either via LAN or using saved HTML pages for read-only access when not at home.
Bottle uses no dependencies other than standard library and the whole framework
is packaged into a single file. It was perfect for the portable app scenario.

After the development started and I saw how difficult it is to make a web
application, I've decided there is no point to confine the result of all that
labor to a single computer. Why use static (maybe even outdated) HTML dumps on
the smartphone when we could access full functionality of the application via
web site?

### Traditional web application

The development was already going on, powered by Python and Bottle. What I was
missing was the server to host my application at. Turns out launching a web site
can be very cheap these days! Even as cheap as free.

I've started with a free domain name from [Freenom] and a free hosting from Red
Hat's [OpenShift]. Both of them have had upsides and downsides but no downsides
were significant enough to turn down the price of free. Freenom domains are
considered low quality from SEO standpoint, but that was completely irrelevant
in my case. OpenShift server had to receive at least one http request every 24
hours to stay awake, and I've cheated that with a cron job on my router. No
inconveniences whatsoever.

I have to digress to emphasize: OpenShift was great! It was easy to set up and
very convenient to use. Its documentation was very thorough and up to date.
OpenShift has introduced me to some concepts I would have not encountered
otherwise, like automated deploying after git push and handling proper wsgi
server. And all of that was for free. I'm very thankful to Red Hat for that.

Unfortunately, as Mr. Heinlein [used to reiterate][TANSTAAFL], there ain't no
such thing as a free lunch. Red Hat could not pay the bills for all the computer
enthusiasts forever and version 3 of OpenShift imposed much tighter restrictions
on the free accounts. I've had to move to a cheap virtual private server and to
learn to set up and maintain an Apache instance on my own. New VPS was sometimes
too slow compared to what OpenShift has offered and I might need to migrate to
another hosting provider later, but everything was fine for now.

I could concentrate on the development.

## The unknown unknowns

Little did I know that while I was keeping myself busy with seemingly important
problems such as whether to store images as blobs in database or as files on the
file system a number of much more important problems were creeping up behind me.
Donald Rumsfeld has called these things *the unknown unknowns* - the things that
we don't know while remaining unaware about the very fact of not knowing.

### ORM

SQLite to store data. Web framework to interact with user. Python to glue them
together. The need in all these parts comes naturally, you could not "forget" to
use any one of them. Yet they are not the only parts that are needed.

Nowhere in Python's database API documentation it says that composing each query
individually is highly inefficient in a database-driven application. No one
hints that there exist a whole other class of libraries called Object-Relational
Mappers ([ORM]). And I was not clever enough to deduce that on my own.

But I wasn't too dumb either. I figured that repeating myself every time I
needed to run a simple query was wrong. So I stashed that code away into [SQL]
class. I figured that Book and Author objects require a lot of common methods to
interact with database. So I separated that code into [TableEntityWithID] class.
I have essentially implemented an ORM without knowing what ORM is. Of course, it
is far inferior to SQLAlchemy and the likes. Of course, I will never use it in
another project now that I know that there are industry standard solutions in
this area. But I see no point in replacing my (suboptimal) implementation now.
Because it works and nothing is broken and nothing is missing.

I consider my ORM to be a valuable educational experience even if it somewhat
stalled the overall progress of the project.

### Multi-threading

While I was developing the application I was using Bottle's builtin wsgiref web
server. Running that server in production environment is not recommended because
it was not built to withstand all the nastiness of open Internet. It is also a
single threaded server which means it can not accept a request until it's done
serving the previous one. The single-threadiness did not concern me - my app was
intended to be used by two or three users tops and simultaneous requests would
happen very seldom. Yet all the proper web servers (Apache, Nginx etc) are
created to scale up to thousands and millions of users. Of course, they are
multithreaded and multiprocessing. My application was not ready for that. Some
would say SQLite is not ready for that either.

At first I considered crippling Apache to use only a single thread or switching
to another server that can operate in a single thread mode. OpenShift seduced me
with maintenance-free setup of Apache and I've abandoned that idea.

The code I've come up with to use a different instance of some objects for each
thread is [rather ugly][ThreadItemPool]. And it works only 90% of the time. So I
just restart the Python workers every hour to avoid HTTP 500 errors when the
database gets locked up. Not the best engineering decision of mine.

There probably exists a proper solution to multithreaded SQLite access, but so
far I have not guessed what keywords to ask Google for it.

### Database migrations

I was aware that changing database schema after the database was populated is
hard and can lead to data loss. So I have put a lot of thought into designing
it. I've even fired up GraphViz and created a [nice chart][schema] that I've
printed out and looked at on the commute. I wanted the database schema to be
iron-clad and to require no changes in future.

I was very naive.

Of course it would require changes. Everything around us changes, we change, our
expectations towards software change. At that point I understood that updating
database manually was not a solution: there would be no trace left whether the
schema was updated, which changes were introduced and which version does it
correspond now to. So I've coded a simple [transition] module that would do that
work for me. And later I've learned that the process I was automating is called
[database migration], and there are tools written by professionals for it.

### JavaScript is a lot of work

I tried to avoid adding dependencies whenever I could. So I've decided to write
what little client-side logic I needed in pure JavaScript. I have no experience
with any JS framework so I thought that avoiding to learn one would compensate
for the inconvenience. I'm not sure it did.

Most of my JS code is written intuitively, with no awareness of best practises
and is filled with "code smells". Writing from scratch was sometimes quite
educating, e.g with AJAX calls - now I understand and can explain them better than
ever before. Overall conclusion is that JavaScript is hard. And that frameworks
probably exist for a reason. If I'll do another web app project, I'll probably
look into some lightweight framework to save time writing JS, which I find not
very enjoyable.

### Modular design

One might think that splitting code into modules comes naturally. At least I
did. And I was wrong. When I was not specifically thinking about their size some
modules tended to grow large. A thousand lines is pretty hard to do a quick
overview on, and even harder to split after the fact. Some modularity has to be
designed from the beginning.

I don't know how I could've avoided that. Guess it comes with the experience.

### MVC

This is another example of a sensible principle that's hard to come by without
someone teaching you. [Model-View-Controller][MVC] is a logical extension of
modularity principle which I was totally unaware about. I have made some
intuitive steps in the right direction, but overall my application is a soup of
interleaved components. That complicates maintenance and further development and
makes the code more difficult to understand for other developers.

If I would've gone with a bigger framework like Flask or Django, MVC mindset
might have been forced down on me. For a newbie who doesn't know anything some
dictatorship isn't that bad.

## It works!

After all the difficulties and complications (both expected and unexpected) I
can proudly say that the Library project is a success. The website is up and
running for almost a year. There has been no significant downtimes and no data
loss, most of our books have been catalogued. And most important, me and my wife
do enjoy using it!

The application supports:

- Creating and editing book entries.
- Storing and displaying book metadata, cover thumbnail and arbitrary related
  files. Each book can be connected with any number of authors, series and/or
  tags.
- Using ISBN to fetch book information from several third-party sources
- Queuing ISBNs for input. This is helpful when you process a lot of books with
  barcode scanner and don't have the time to clean up automatic metadata on each
  one of them
- Adding 1-to-5 star ratings and text reviews to any book in the library
- Searching for books by exact metadata match and with wildcards

You can access [the source code][source] on GitHub and see the site in action at
<https://morebooks.ml> (registration is for family members only, sorry).  Of
course, there are plenty of improvements to be made (you can see how long the
TODO list is), but the maintenance itself requires almost zero attention now
and I can happily switch from being a developer to becoming the end user.

[Bottle]: https://bottlepy.org/
[Freenom]: https://freenom.com/
[MVC]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[ORM]: https://en.wikipedia.org/wiki/Object-relational_mapping
[OpenShift]: https://www.openshift.com/
[SQL]: https://github.com/sio/HomeLibraryCatalog/blob/1452531ec05f049c6a758530d7f526f05c188ba1/hlc/db.py#L356
[TANSTAAFL]: https://en.wikipedia.org/wiki/The_Moon_Is_a_Harsh_Mistress
[TableEntityWithID]: https://github.com/sio/HomeLibraryCatalog/blob/1452531ec05f049c6a758530d7f526f05c188ba1/hlc/items.py#L37
[ThreadItemPool]: https://github.com/sio/HomeLibraryCatalog/blob/1452531ec05f049c6a758530d7f526f05c188ba1/hlc/web.py#L1173
[blog]: http://www.zackgrossbart.com/hackito/the-library-problem/
[database migration]: https://en.wikipedia.org/wiki/Schema_migration
[relational model]: https://en.wikipedia.org/wiki/Relational_model#History
[schema]: https://github.com/sio/HomeLibraryCatalog/blob/master/docs/relations.gv.pdf
[source]: https://github.com/sio/HomeLibraryCatalog
[transition]: https://github.com/sio/HomeLibraryCatalog/blob/master/hlc/db_transition.py
