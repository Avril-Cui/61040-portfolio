# Exercise 1: Reading a concept

## Q1
Invariants. What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

There are two invariants of the state.

### Invariant 1
At any time, for every Request in a Registry, the remaining count is never negative (`Request.count` $\geq 0$).

The action whose design is most affected by this invariant is the action `purchase (purchaser: User, registry: Registry, item: Item, count: Number)`. This action has a `requires` clause that specifies "registry exists, is active and has a request for this item with at least count" and a `effects` clause that decrements the count in the matching request. Since this action requires the Request for this `item` has at-least the asked amount of counts by the purchase, the `count` for the corresponding Request is still $\geq 0$. Therefore, the invariant is preserved.

I think this invariant is the most important, because the system would be problematic if many buyers make more purchases that what the Requests ask for, contradicting the purpose for having a registry system. The buyers would have spend this money on purchasing other available Requests. There would then a many waste of resources if this invariant is not preserved.

### Invariant 2
Every Purchase belongs to exactly one Request, and this Purchase's' `Item` must matches the corresponding Request’s `Item`.

// ONLY THE MOST IMPORTANT ONE HERE


There are several actions whose design are most affected by this invariant:
1. `purchase` has a `requires` statement that checks there exists a Request for this item. Thus, it ensures every new, valid Purchase must have a matching Request with the same `item`, preserving the invariant.

2. `addItem` has an `effects` clause stating that "if a request for this item exists, add the count; otherwise create a new request for the item with this count and add to registry." This clause ensures that no two Requests will have the same `item`, because the second `addItem` call will simply add the `count` of the first created Request. This, the action ensures every Purchase will only have one Request with a matching `Item`, preserving the invariant.

3. `removeItem` must remove the corresponding Request as well as any contained/matched Purchases so that no Purchase is left unattached. In this way, we preserve the invariant.

## Q2
Fixing an action. Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

`purchase (purchaser: User, registry: Registry, item: Item, count: Number)` does not require the `count` to be positive. If `count` is negative, then due to Invariant 1, the count for the corresponding Request is always greater than the count for this "negative" Purchase. However, this negative Purchase is meaningless and would increase the `count` for the corresponding Request (since decrement by a negative number means to increase), potentially leading to oversell during later Purchases since the Request didn't meant to ask for this many `count`. This problem is the case for an "unsafe design" mentioned in lecture, where we have a transition from good states to some bad state.

A fix would be tightening the requirement to `requires count > 0` for `purchase`. This ensures that the inductive step holds. Given `counts` that are non-negative before the action, they remain so after the action because the only change is a decrease by a positive amount after tightening the requirement. 

## Q3
Inferring behavior. The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

Yes, repeated opening and closing is allowed. Since the only requirements for `open (registry: Registry)` are the registry exists and it is not active, a registry that was closed also satisfies this precondition. Similarly, once a registry is open, it also satisfies the precondition (i.e., registry exists and is active) for `close (registry: Registry)` Therefore, we can have $open\to close\to open\to close...$ arbitrarily many times.

The reason why we allow this is to let the recipient pause public visibility of their registry without completing deleting/destroying the registry. For example, they might want to pause visibility if they want to edit the items or fix any mistakes, then reopen when they are ready. The repeated opening and closing keeps the registry's history, making it more convenient for the recipient.

## Q4
Registry deletion. There is no action to delete a registry. Would this matter in practice?

`close` is a substitute version of delete. However, it would matter in practice, because if users no longer want a registry, the only thing they can do is to `close` this registry. This means that these registries that users no longer want eventually accumulate in our database, taking up extra spaces, which is an extra cost for data storage in practice. 

## Q5
Queries. What are two common queries likely to be executed against the concept state? (Hint: one is executed by a registry owner, and one by a giver of a gift.)

SKIP

## Q6
Hiding purchases. A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

First, we augment the **state**, in particular, the Requests:

a set of Registrys with \
    an owner User \
    an active Flag \
    a set of Requests \
    a surpriseMode Flag 

We also augment the **actions** by adding two actions:

setSurpriseMode (registry: Registry) \
    **requires** registry exists and it is not on surpriseMode \
    **effects** made registry surpriseMode

removeSurpriseMode (registry: Registry) \
    **requires** registry exists and it is on surpriseMode \
    **effects** made registry not surpriseMode

## Q7
Generic types. The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

Using generic parameters means we have polymorphic type passing in and out of actions. This is preferable to representing items with their names because in this way, our concept assumes nothing about these parameters, and they are just opaque references to any kind of users and any kind of items. This enables reusability and generalization, and we won't need to worry about minor details like changes in the specific item representations.

It also offers a quick and handy way to query for items, since we won't need to check if all properties like names, descriptions, prices match up.

# Exercise 2: Extending a familiar concept
## Q1 and Q2
**concept** PasswordAuthentication

**purpose** limit access to known users

**principle** after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user

**state**

a set of Users with

    a username String
    a password String

**actions**

    register (username: String, password: String): (user: User)
        **requires** no user exists with this username
        **effects** create and return a new user with this username and password

    authenticate (username: String, password: String): (user: User)
        **requires exists a user with this user name and password
        **effects* return that user u

## Q3
The invariant for the state would be there is at most one user with a given username. 

The initial state (base case) is good, because by default, we have an empty set of Users. So, the base case is trivial and the invariant holds for the base case.

During the inductive steps (i.e., during transitions made by the actions), only the **register** action can possibly break the invariant, so we need to guard the **register** action. The precondition "no user exists with this username" ensures that one username will not get registered twice to create two different users. **authenticate** action does not mutate the state, so it will not violate the invariant. Since the initial state is good and no action can violate the invariant (i.e., turning a good state into a bad state), the invariant is preserved.

## Q4
We augment the state and actions.

**state**

a set of Users with

    a username String
    a password String
    an email String
    a confirmed Flag

a set of Confirmations with

    a user User
    a token Secret

**actions**

    register (username: String, password: String, email: String): (user: User, token: Secret)
        **requires** no user exists with this username
        **effects**
            create a new user with this username and password and not confirmed
            create a new confirmation for the user with with a new Secret token
            return the user and the Secret token
        
    confirm (username: String, token: Secret)
        **requires**
            exists a user with this username and
            the user is not confirmed and
            exists a confirmation for this user with the matching Secret token
        **effects**
            make the corresponding user confirmed

    authenticate (username: String, password: String): (user: User)
        **requires exists a user with this username and password
        **effects* return that user

**invariants**
1. At most one user with a given username
2. The Secret token for each Confirmation will not get reused. This also means each Secret token belongs to exactly one user.

# Exercise 3: Comparing concepts

**concept** PersonalAccessTokenClassic [User, Item]

**purpose** \
    enable programmatic access on behalf of a user without sharing their password

**principle** \
    a user generates a token with a chosen scope; \
    the system returns the token's secret once; \
    the user can authenticate using the token; \
    the token can be deleted, after which it can not authenticate;

**state**
    a set of Tokens with
        an owner User
        a secret Secret
        a scope Scope
        a revoked Flag
        an expiration Time
    
    a set of Users with
        an username String

**actions**
    createToken (owner: User, scope: Scope, expiration: Time): (token: Token, secret: Secret):
        <!-- requires owner exists (need a owner state) -->
        **effects**
            create a new token with this owner;
            set the scope as the provided or default scope, 
            set revoked to false,
            set a newly generated secret string,
            set expiration as the provided or default time
            return the token's secret
    
    deleteToken (owner: User, token: Token)
        **requires** token exists and the owner of the token matches the action's input owner and deleted is false
        **effects** make deleted true
    
    authenticateWithToken (username: String, secret: Secret) : (user: User)
        **requires**
            a token exists with a secret matching the action's input secret,
            and the owner of this token has the matching username,
            and deleted is false for this token,
            and the current timestamp is before expiration for this token
        **effects** authentication passed, return that owner

Additionally, there is an important invariant for the state:
1. Each secret belongs to exactly one token

## Difference with PasswordAuthentication
For PersonalAccessToken, one user can be the owner for multiple tokens at the same time, each with its own scopes and expiration. User can also revoke the tokens individually. This differentiates PersonalAccessToken from PasswordAuthentication. For PasswordAuthentication, each user would only have a single password. Consider the following different use cases:
- PasswordAuthentication: log in to the web UI, authenticate as the user for a session to change profile, set up billing information, or interact as a "user" (for example, posting under this user profile)
- PersonalAccessToken: create a new token with different scopes and expirations for each server, app, or bot. This allows per-integration or per-action permission isolation. PersonalAccessToken would be especially good for dev (and this is why GitHub implements it) because of its environment isolation and separation.

## GitHub page suggestions
- Avoid terms like "personal access tokens are like passwords" (which were mentioned several times through out the article). Focus on describing the concept of personal access tokens.
- Add a comparison section at the bottom to list the different use cases for password authentication and personal access token (like what I did above). Focus on why GitHub uses personal access token, including elaborating on uses cases where "scope" and "expiration" are important.


# Exercise 4: Defining familiar Concepts

## URL Shortener
Define a concept for the essential function of a URL shortening service such as tinyurl.com or bit.ly. Your concept should support both user-defined and autogenerated URL suffixes.

**concept** UrlShortener [User, Item]

**purpose** provide short aliases that redirect to long target URLs for convenience

**principle**
    user chooses a suffix or asking the system to generate one,
    the system ensures the suffix is unique for the domain and returns the short link,
    user navigates to the long target URL's location by clicking on the short link,
    user can deactivate the link so it no longer redirects

**state**
a set of Links with
    a Domain
    a Suffix
    a target URL
    an owner User
    an active Flag

**actions**

createCustom (owner: User, domain: Domain, suffix: Suffix, target: URL) : (link: Link)
    requires no link exists with the same (domain, suffix) pair
    effects create a new link with this owner, domain, suffix, and target, and set active to true

createAuto (owner: User, domain: Domain, target: URL): (link: Link)
    effects
        autogenerate a fresh suffix not used this is domain,
        create a new link with this owner, domain, autogenerated suffix, and target, and set active to true

redirect (domain: Domain, suffix: Suffix) : (target: URL)
    requires exist a link with matching (domain, suffix) pair and has active set as true
    effects return the target

deactivate (owner: User, link: Link)
    requires
        link exists
        the owner of this link matches the provided owner
        this link is active
    effects
        make active false for this link

The key invariant for the state is there is at most one link per (domain, suffix) pair. The base case (when there is an empty set of Links) is trivial and holds the invariant. We preserve the invariant by:

1. Having a requires statement for `createCustom` that forbids creating shortened URL when the custom (domain, suffix) pair is occupied

2. Having `createAuto` autogenerate fresh/unoccupied suffix to prevent having colliding (domain, suffix) pair.

Since we start with a good base case, and no transition/action would turn a good state into a bad state, the invariant is preserved.

## Conference Room Booking

Define a concept for the essential function of a service for booking conference rooms in a company or university department, like CSAIL’s room booking system. Note: you do not need to include recurring bookings.


**concept** ConferenceRoomBooking [User, Item]

**purpose** let people reserve public rooms at specific times to avoid conflict

**principle**
    an admin creates availability slots for rooms with start and end times
    a user reserve an available slot for a room
    a reserved slot cannot be double-booked
    a user can cancel their reservation
    an admin can deactivate slots

**state**
    a set of Bookings with
        a User
        a Slot
    
    a set of Slots with
        a Room
        a Time
        an active Flag
    
    a set of Rooms with
        a capacity Number

**actions**

    addRoom (capacity: Number): (room: Rooms)
        **effects** make room with this capacity

    createSlot (room: Room, time: Time): (slot: Slot)
        **requires**
            room exists \
            input start is before end \
            no slot exists with the same room and time
        
        **effects** 
            create a fresh slot for this room and time, and set active as True
        
    deactivateSlot (slot: Slot)
        **requires** slot exists and slot is active
        **effects** set slot as not active
    
    reserve (user: User, slot: Slot, capacity: Number): (booking: Booking)
        **requires**
            slot exists and slot is not active
            no booking exists for this slot
            the user requested capacity is less than the capacity of the room for this slot
        
        **effects**
            create a new booking for this user and slot
    
    cancelReserve (user: User, booking: Booking)
        **requires** booking exists and has the matching user
        **effects** remove booking

There are two core invariants that needs to be preserved:
1. There is at most one slot per (room, time) pair. The base case is good under this invariant because we initially have an empty set of slots. We guard this invariant by the requires statement for `createSlot`, which explicitly requires "no slot exists with the same room and time."

2. There is at most one booking per slot, avoiding double-booking. The base case is good under this invariant because we initially have an empty set of booking. We guard this invariant by the requires statement for `reserve`, which checks "no booking exists for this slot."

Since we start with a good base case for both invariants, and no transition/action would turn a good state into a bad state, the invariants are preserved.

## Billable Hours Tracking

**concept** BillableHoursTracking [User, Item]

**purpose** record billable work sessions per employee per project

**principle**
    an employee starts a session by choosing a project and entering a short work description;
    later this employee ends this session;
    if a session exceeds its  (if any) plannedEnd time or some default endDuration after start, the system automatically ends the session and marks it as auto-ended

**state**

    a set of Sessions with
        an Employee
        a Project
        a start Time
        an end Time (optional)
        a plannedEnd Time (optional)
        a description String
        an autoEnded Flag

    a endDuration Duration

**actions**

    startSession (employee: Employee, project: Project, description: String, start: Time, plannedEnd: optional Time): (session: Session)
        **requires**
            no session exists for this employee with the same (project, start)
        **effects**
            create a new session $s$ with this employee, project, start, description
            set end for $s$ as None and autoEnded as false
            if plannedEnd is provided, set plannedEnd
            

    endSession (employee: Employee, session: Session, end: Time):
        **requires**
            session exists with this employee and has end as None
            input end is after session's start
        **effects** set session's end as this input end
    
    autoEnd (session: Session, current: Time):
        **requires**
            session exists
            session has end as None
            current is after session.start
            (
                (session.plannedEnd is present and current is after session.plannedEnd)
                OR
                (current is over endDuration after session.start)
            )
        **effects**
            set end for the session as current
            set autoEnded as True

I have implemented several states and actions to handle a case in which someone forgets to end a session.

First, each session has an optional `plannedEnd` time, which the employee can set when they perform `startSession` action. The system will automatically end any session that exceeds its plannedEnd time . If no plannedEnd is set for a session, the system will automatically end the session if it exceeds the system's `endDuration`. Therefore, no session will last longer than `endDuration`.

Additionally, to differentiate between manually-ended and auto-ended sessions, each session as an `autoEnded` flag, which is true if the session is automatically ended. This information would be helpful when companies are using the tracked record to decide billing.





 



