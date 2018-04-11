I have a growing opinion that I think is highly controversial in some circles: We have become too obsessed with objects and "object design." We are surrounded by Objects everywhere: Design Patterns, Domain-Driven Design, UML, etc. We think objects make our world better. But what if they are actually making our software more coupled, our code another step removed from the data, our systems harder to integrate, and incurring all sorts of versioning problems along the way?

> "I invented the term object-oriented, and I can tell you that C++ wasn't what I had in mind"
>
> Alan Kay

# Our worldview is shaped by the tools we use
TODO (how C++/Java/C# encourage us to be "object-first" in thinking)

TODO (the world we are in: system integration is hard (data manipulation into specific "shapes"), versioning is hard (semVer doesn't actually help), we have problems with composition from the bottom all the way to the top).

> I'm sorry that I long ago coined the term "objects" for this topic because it gets many people to focus on the lesser idea. The big idea is "messaging"...
> The Japanese have a small word - ma - for "that which is in between" - perhaps the nearest English equivalent is "interstitial". The key in making great and growable systems is much more to design how its modules communicate rather than what their internal properties and behaviors should be.
>
> Alan Kay ([AlanKayOnMessaging](http://wiki.c2.com/?AlanKayOnMessaging))

TODO (having "ma" everywhere in our programs but it's completely implicit and hard to reason about; where this non-data-first approach gets in our way: function parameters / assumptions about data structure vs. data contents, dependencies between components: the types are the "hidden" contract (should I need to care about extraneous stuff given to me or what structure it is in when given to me, or should I just be able to get the data I care about?), web services (accepted types, JSON schema for validation, etc.); microservices defined around this "object" boundary)

TODO (The containers, objects, are not Abstractions, they are "Concrete"-ions)

TODO (With a focus on the container itself, we lose perspective on the data itself; RDF example of a customer name across multiple objects)

TODO (Limits reusability of code, makes code harder to change; change one type/object and tests break; change one type/object and contracts with all other code changes)

TODO (not out-of-left field; idiomatic Clojure does some of this with its focus on maps, especially powerful when combined with persistent data structures)

TODO (any way to insert Linda Tuplespaces here?)

TODO (types as contracts and their impact on versioning; looking up data via URL (imagine keys in a dictionary where the keys are URLs) versus a smaller name or string of something, for compiled languages may even be based on an offset into a v-table that describes the object; similar to "late-binding in all things" (find reference from Alan Kay))

TODO (when wanting to achieve better function composition, are separated parameters the moral equivalent to looking up specifics rather than having one parameter where values are looked up via key/URL? "Parameter-driven programming" can make function composition more difficult; what do you do if you want to pass along some sort of "context" where a value from an earlier fn call can be used later down the sequential chain?)