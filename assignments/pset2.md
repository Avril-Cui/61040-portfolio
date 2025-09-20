# 1. Concept Questions
## Contexts
The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

The contexts in the NonceGeneration concept is used to define which scope we are requiring uniqueness for. In particular, instead of requiring every generated string to be unique across the entire system, uniqueness only needs to hold within a given context.

In an URL shortening app, a context will be a domain or a base URL under which the shortened links are created. For example:
- Within tinyurl.com, all nonces must be unique. We cannot have two tinyurl.com/abc
- The same nonce (/abc) can exist under two contexts without conflict. We can have both tinyurl.com/abc and mydomain.com/abc

## Storing using strings
Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

It is necessary for NonceGeneration to store sets of used strings because the concept maintains an invariant that each generated nonce must be a string not used before for that context. To enforce this invariant, the context must remember ALL the strings it has already generated. So, we need some way to store or note those used strings for each context.

A simple implementation could just maintain a counter for each context that starts at 0 and increments with each `generate` call. The counter acts as the way to "remember used strings." Each time when the system is asked to generate a new nonce (i.e., when `generate` is called), it will:

1. Convert the current counter value into a string (the easiest way is $0\to$ "0", $1\to$ "1", ...)
2. Increment the counter for the context


So, the set of strings in the specification is implemented with the counter.

## Words as nonces
One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?

### Advantage
It is much easier to remember, say, and type shortened URL with dictionary words as suffixes. Usually, people want to shorten URL so that they can share this link with others more easily. A short link with dictionary words suffixes makes the link more human-readable and shareable in conversation.

### Disadvantage
There is a limited number of short and common dictionary words. If the system runs out of short and simple words, it could use long, complex words (e.g., sesquipedalian). These words can be difficulty to read out, type out, or remember exactly.

# NonceGeneration modification
We modify NonceGeneration to realize this idea.

```
concept NonceGeneration [Context, Dictionary]
purpose
    generate unique string of a dictionary word within a context
principle
    each generate returns a word or word phrase not returned before for that context
state
a set of Contexts with
    a used set of Strings
    a dictionary Dictionary
actions
    generate (context: Context) : (nonce: String)
        requires: exists at least one unused phrase in the dictionary
        effect: returns a word from the dictionary that is not already used by this context
```




