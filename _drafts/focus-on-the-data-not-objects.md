
TODO (Alan Kay quote on OOP)

TODO (The containers, objects, are not Abstractions, they are "Concrete"-ions)

TODO (With a focus on the container itself, we lose perspective on the data itself; RDF example of a customer name across multiple objects)

TODO (Limits reusability of code, makes code harder to change; change one type/object and tests break; change one type/object and contracts with all other code changes)

Taken from Slack conversation from me to Howard:
> And dude, just watched Rich's "Effective Clojure" talk again. I'm very intrigued by Clojure's approach to "return a map from everything", the inspiration from RDF, and its impact on reasoning about the larger system, the pieces in between, and composing functions as well.
> That striving to get to the point where even the data structures our functions deal with are semantically aware through stuff similar to RDF (Rich uses namespace pathing as the URL equivalent combined with ns-qualified keywords in a Map).
> And his point about how most "abstractions" in OOP are not actually "abstract"-tions, they are "concrete"-tions. In that world, you promote the container itself above all else instead of focusing on the data itself.