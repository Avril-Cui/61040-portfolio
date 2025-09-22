# Part One: Concept Questions
## 1. Contexts
The contexts in the NonceGeneration concept is used to define which scope we are requiring uniqueness for. In particular, instead of requiring every generated string/nonce to be unique across the entire system, uniqueness only needs to hold within a given context.

In an URL shortening app, a context will be a domain or a base URL under which the shortened links are created. For example:
- Within tinyurl.com (a context), all suffixes/nonces must be unique. We cannot have two tinyurl.com/abc
- The same suffix/nonce (/abc) can exist under two contexts without conflict. We can have both tinyurl.com/abc and mydomain.com/abc, because they are under different domains.

## 2. Storing using strings
It is necessary for NonceGeneration to store sets of used strings because the concept maintains an invariant that each generated nonce must be not used before for that context. To enforce this invariant, the NonceGeneration concept must remember or store ALL the strings it has already generated for that context.

A simple implementation could just be maintaining a counter for each context that starts at 0 and increments with each `generate` call. The counter acts as the way to "remember the set of used strings in this context," thus preserving the invariant. The abstraction function is as follows:

1. Suppose for each context $C$, we have `counter[C]` storing the integer count in the implementation for context $C$.
2. The abstraction function maps this counter to the set of used strings as as: $AF(counter[C])=\{encode(i)\ |\ 0\leq i< counter[C] \}$, where `encode(i)` is some function that turns an integer into a string.
    - A simple example of `encode(i)` is the function that converts an integer to its string form, i.e., mapping $0\to$ "0", $1\to$ "1", ...

 <!-- Each time when the system is asked to generate a new nonce (i.e., when `generate` is called), it will:

1. Convert the current counter value into a string (the easiest way is $0\to$ "0", $1\to$ "1", ...)
2. Increment the counter for the context
 -->

<!-- So, the set of strings in the specification is implemented with the counter. -->

## 3. Words as nonces
From the perspective of the user:
### Advantage
It is much easier to remember, read, share, and type shortened URL with dictionary words as suffixes. Usually, people want to shorten URL so that they can share and access this link more easily.

### Disadvantage
There is a limited number of short and common dictionary words. Very quickly (especially if the URL shortening app is used by many people), the system will run out of words, and the URL shortening app will no longer work for users. So there is a very limited amount of generations users can request, leading to inconveniences and a bad user experience.

 <!-- meaning the same suffix will be generated twice for different targetUrls, which is misleading and could cause conflicts when users try to navigate to their targetUrl with the shortened URL (i.e., the shorten URL might redirect the user to the wrong target URL). -->

Also, if the system runs out of short and simple words, and to avoid collision, it could start using longer, more complex words (e.g., sesquipedalian). These words can be difficulty to read out, type out, or remember exactly.

### NonceGeneration modification
We modify NonceGeneration to realize this idea.

```
concept NonceGeneration [Context, Dictionary]

purpose
    generate unique string of a dictionary word within a context

principle
    each generate returns a dictionary word not returned before for that context

state
    a set of Contexts with
        a used set of Strings
        a dictionary Dictionary // i.e., a set of common dictionary words
    
actions
    generate (context: Context) : (nonce: String)
        requires: exists at least one word in the dictionary that is not included in the used set of Strings
        effect: returns a word from the dictionary that is not already used by this context, and add this word to the used set of Strings
```

# Part Two: Synchronizations for URL Shortening

## 1. Partial matching
In the `generate` sync, its purpose is to ask NonceGeneration to make a unique nonce in the given context. Therefore, the only argument needed to generate this nonce is the shortUrlBase, which will be the context. The targetUrl isn't used in NonceGeneration at all, so it is irrelevant and is omitted to preserve the separation of concerns.

In the `register` sync, our purpose is to register a shortenUrl given a shortUrlBase and a shortUrlSuffix (nonce), and associate this shortenURL with a targetUrl. So, we need the full set of information for registration, which is why we need both the shortUrlBase and targetUrl in the request.

## 2. Omitting names
The omitting name convention only works if it is unambiguous that the argument name and the bound variable name are exactly the same. 

In other cases, to maintain modularity and separation of concerns, we could use different names in different concepts when referring to the same item/variable. For example, a `nonce` generated by NonceGeneration and a `shortUrlSuffix` in UrlShortening are the same thing in the `register` sync (i.e., we pass in nonce as a shortUrlSuffix to UrlShortening.register). If we follow the convention and omit the names, we will result in ambiguity (i.e., which name do we use? nonce or shortUrlSuffix?) and lost track of the connection between variables like nonce and shortUrlSuffix when trying to associate them in a sync.

By keeping the explicit name: variable format, we maintain the modularity goal of keeping responsibilities explicit while making it clear that two concept interact. This explicit form also avoid confusion about which value is bounded to which variable.

## 3. Inclusion of request
The sync `generate` is triggered by a user request to shorten a URL. Similarly, the sync `register` is trigged by a user request to shorten URL, but now it also waits until NonceGeneration has produced a nonce. Both syncs exist to respond to some external request from a user.

On the other hand, the sync `setExpiry` is handled by the server and is triggered by the completion of another concept's action (i.e., UrlShortening.register). No user request is needed because we want the `setExpiry` sync to be an automatic system follow-up after any URL is registered. This is why we do not include the `request` action as it is not associated with a user action or request.

## 4. Fixed domain
We modify the `generate` and `register` syncs. There is no need of passing in ShortUrlBase as a variable input in the request. We fix it to some predefined value. For demonstration, we set it as "bit.ly":

```
sync generate
    when Request.shortenUrl ()
    then NonceGeneration.generate (context: "bit.ly")

sync register
    when
        Request.shortenUrl (targetUrl)
        NonceGeneration.generate (): (nonce)
    then UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: "bit.ly", targetUrl)
```

setExpiry sync will remain the same.

## 5. Adding a sync
```
sync expireShortUrl
    when ExpiringResource.expireResource (): (resource)
    then UrlShortening.delete (shortUrl: resource)
```

# Part Three: Extending the design
## 1. Additional Concepts
We first define some additional concepts.

### ShortUrlOwnership 
```
concept: ShortUrlOwnership [User]

purpose:
    record which user registered each resource

principle:
    after a resource is registered, its owner is the user who registered it, and only that user can access or change that resource

state:
    a set of Ownerships with
        a resource Resource
        an owner User

actions
    assign(resource: Resource, owner: User)
        requires: no ownership exists for resource
        effect: records ownership to associate resource with owner

    getOwner(resource: Resource): (owner: User)
        requires: ownership exists for resource
        effect: returns the owner of the given resource
    
    authorize(resource: Resource, user: User): (verified: Boolean)
        requires: ownership exists for resource and owner matches the given user
        effect: approves authorization, return verified as True
```

### AnalyticsCounts
```
concept: AnalyticsCounts [Resource]

purpose:
    provide analytics to track how many times each resource is accessed

principle:
    after initializing a counter for a resource, each successful access to this resource increase its count

state
    a set of Counters with
        a resource Resource
        a count Number
    
actions
    initCount(resource: Resource)
        requires: no counter exists for resource
        effect: creates a counter for resource with count = 0
    
    increment(resource: Resource)
        requires: counter exists for resource
        effect: increase count of resource by 1
    
    getCount(resource: Resource): (count: Number)
        requires: counter exists for resource
        effect: return the current count for the resource
```

## Synchronizations
Next, we specify three important synchronizations with the new concepts.

### When shortening are created
```
sync initAnalytics

when:
    Request.shortenUrl(user)
    UrlShortening.register (): (shortUrl)

then:
    // assign the ownership of the shortUrl
    ShortUrlOwnerShip.assign (resource: shortUrl, owner: user)
    // init the counter of the shortUrl
    AnalyticsCounts.initCount (resource: shortUrl)
```

### When shortening are translated to targets
```
sync recordAccess

when:
    Request.accessShortUrl (shortUrl)
    UrlShortening.lookup (shortUrl)

then:
    AnalyticsCounts.increment (resource: shortUrl)
```

### When a user examines analytics
```
sync viewAnalytics

when:
    Request.viewAnalytics (shortUrl, user)
    ShortUrlOwnership.authorize(resource: shortUrl, user) : (verified)
        requires: verified is True

then:
    AnalyticsCounts.getCount(resource: shortUrl): (count)
```

## Additional features
Finally, we outline how each feature requests will get realized.

### Allowing users to choose their own short URLs
We can achieve this by adding a new `registerCustom` sync, and a new action `customSuffix` under NonceGeneration to validate the user-inputted nonce before registration:
```
// in NonceGeneration

actions
    customSuffix (context: Context, nonce: String): (verified: Boolean)
        requires: nonce is not already used by this context
        effects: passes verification, add nonce to context, return verified as True
```

```
sync registerCustom

when:
    Request.shortenUrlCustomSuffix (targetUrl, shortUrlBase, shortUrlSuffix, user)
    NonceGeneration.customSuffix (context: shortUrlBase, nonce: shortUrlSuffix) : (verified)
        requires: verified is True

then:
    UrlShortening.register (shortUrlSuffix, shortUrlBase, targetUrl)
```

## Using the “word as nonce” strategy to generate more memorable short URLs
We generate a new concept called `WordNonceGeneration`, like how we specify it in Part 1 previously:

```
concept WordNonceGeneration [Context, Dictionary]
purpose
    generate unique string of a dictionary word within a context
principle
    each generate returns a dictionary word not returned before for that context
state
a set of Contexts with
    a used set of Strings
    a dictionary Dictionary
actions
    generate (context: Context) : (nonce: String)
        requires: exists at least one word in the dictionary that is not included in the used set of Strings
        effect: returns a word from the dictionary that is not already used by this context, and add this word to the used set of Strings
```

Now, we can have syncs `generateMemorable` and `registerMemorable` for more memorable short URLs. They are essentially the same syncs as `generate` and `register` syncs, but we replace NonceGeneration.generate with WordNonceGeneration.generate to support "word as nonce" strategy. We also 

However, this feature has its own advantage and disadvantage, as described in Part 1. So, this feature may not always be desirable.


## Including the target URL in analytics
The way we set up `AnalyticsCounts` concept is very generic and follows the guideline of separation of concerns. Therefore, it is very simple to include target URL in analytics. We simply add three new syncs for target URL analytics:

```
sync initTargetUrlAnalytics
when:
    Request.shortenUrl(targetUrl, user)
    UrlShortening.register (): (shortUrl)
then:
    // init the counter of the targetUrl
    AnalyticsCounts.initCount (resource: targetUrl)

sync recordTargetUrlAccess

when:
    Request.accessShortUrl (shortUrl)
    UrlShortening.lookup (shortUrl): (targetUrl)

then:
    AnalyticsCounts.increment (resource: targetUrl)


sync viewTargetUrlAnalytics

when:
    Request.viewAnalytics (targetUrl)

then:
    AnalyticsCounts.getCount(resource: targetUrl): (count)
```

However, we can see an obvious caveat in the `viewTargetUrlAnalytics` sync. Different users can generate multiple shortUrls that directs to the same targetUrl. This is problematic because the prompt related to analytics specified that: "Analytics should not be public but should be viewable only by the user who registered the shortening." Thus, we probably should not include this feature of viewing target URL analytics due to privacy concerns. We don't want users to view analytics involving other users.

## Generate short URLs that are not easily guessed
To achieve this feature, we simply modify the `generate` action in NonceGeneration so that instead of returning any nonce that is not already used by the context, we return some secure nonce (generated by cryptographic encryption algorithms or more random algorithms) that are not used by this context.

We also see that this feature can contradict with feature 2, because it's difficult to both achieve memorable words and security for nonces.

## Supporting reporting of analytics to creators of short URLs who have not registered as user
I think this feature is undesirable and should not be included. The current analytics concept design is based on the core that "Analytics should not be public but should be viewable only by the user who registered the shortening." Letting non-registered creators view analytics weakens that privacy model and conflates identity authentication with analytics.

