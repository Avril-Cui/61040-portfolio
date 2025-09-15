# Exercise 1: Reading a concept

## Q1
There are two invariants of the state.

### Invariant 1 (counts/aggregation) - most important
At any time, for every request in a registry, the remaining count is never negative (`request.count` $\geq 0$).

I think this invariant is the most important, because we need a system in which the requests' counts reflect the amount of items the recipient still needs, preventing overselling the wishlist. If we don't limit `request.count` to be nonnegative, givers can make more purchases than what the recipient asks for, contradicting the purpose of having a registry system.

The action whose design is most affected by this invariant is the action `purchase (purchaser: User, registry: Registry, item: Item, count: Number)`. This action has a precondition that specifies "registry exists, is active, and has a request for this item with at least count" and a `effects` clause that decrements the count in the matching request. Since this action requires the request for this item to have at least the asked amount of counts by the purchase, the `count` for the corresponding request is still $\geq 0$ after decrementing. Therefore, the invariant is preserved by this action.

### Invariant 2
Every purchase must belong to exactly one request, and this purchase's item must match the corresponding request’s item.

<!-- There are several actions whose design is affected by this invariant:
1. `purchase` action has a `requires` statement that checks that there exists reRequest for this item. Thus, it ensures every new, valid purchase must have a matching request with the same `item`, preserving the invariant.

2. `addItem` action has an `effects` clause stating that "if a request for this item exists, add the count; otherwise create a new request for the item with this count and add to the registry." This clause ensures that no two requests will have the same `item`, because the second `addItem` call will simply add the `count` of the first created request. Thus, this action preserves the invariant. -->

## Q2

`addItem (registry: Registry, item: Item, count: Number)` does not require the `count` to be positive. If `count` is negative, then we can continue to perform `addItem` on the same existing registry and item, which will repeatedly toggle the "effects" statement to add the negative count. This would mean continuously decreasing the count of the request for this item, and eventually we would reach negative count, which breaks Invariant 1. This problem is the case for an "unsafe design" mentioned in lecture, where we have a transition from good states to some bad state.

A fix would be tightening the precondition to `requires count > 0` for `addItem`. This ensures that the inductive step holds. Given `counts` that are non-negative before the action, they remain so after the action because now we can only increase the count, which started as non-negative. So, by induction, Invariant 1 is preserved after the fix. 

<!-- `purchase (purchaser: User, registry: Registry, item: Item, count: Number)` does not require the `count` to be positive. If `count` is negative, then due to Invariant 1, the count for the corresponding Request is always greater than the count for this "negative" Purchase. However, this negative Purchase is meaningless and would increase the `count` for the corresponding Request (since decrement by a negative number means to increase), potentially leading to oversell during later Purchases since the Request didn't meant to ask for this many `count`. This problem is the case for an "unsafe design" mentioned in lecture, where we have a transition from good states to some bad state.

A fix would be tightening the requirement to `requires count > 0` for `purchase`. This ensures that the inductive step holds. Given `counts` that are non-negative before the action, they remain so after the action because the only change is a decrease by a positive amount after tightening the requirement.  -->

## Q3
Yes, repeated opening and closing is allowed. Since the only requirements for `open (registry: Registry)` are that the registry exists and it is not active, a registry that was closed also satisfies this precondition. Similarly, once a registry is opened, it also satisfies the precondition for `close (registry: Registry)`. Therefore, we can have $open\to close\to open\to close...$ arbitrarily many times.

The reason why we allow this is to let the recipient pause public visibility of their registry without completely deleting/destroying their registry. For example, they might want to pause visibility if they want to edit the items or fix any mistakes, then reopen when they are ready. The repeated opening and closing keep the registry's history, making it more convenient for the recipient.

## Q4
`close` is a substitute version of delete, which makes a registry inactive and hidden from public view. However, it would matter in practice because if users no longer want a registry, the only thing they can do is to `close` it. This means that these old, unused registries remain in the system's database indefinitely, taking up extra spaces, which is an extra cost for data storage in practice. 

## Q5
Two common queries likely to be executed against the concept state are:
1. For a registry owner: Which request's items have been purchased by givers, and which givers have made those purchases? Registry owners need to understand this to track the progress of their registry. They also need this information to send thank-you notes to givers later on.

2. For a giver of a gift: Which request's items are still available for purchase, and how many remain? This query helps givers understand which items are needed and in what quantity, so they don't overspend or buy something already covered.

## Q6
First, we augment the **state**, in particular, the Requests set:

```
a set of Registrys with
    an owner User
    an active Flag
    a set of Requests
    a surpriseMode Flag
```

We also augment the **actions** by adding two new actions:

```
setSurpriseMode (registry: Registry)
    requires: registry exists and it is not on surpriseMode
    effects: make registry surpriseMode

removeSurpriseMode (registry: Registry)
    requires: registry exists and it is on surpriseMode
    effects: make registry not surpriseMode
```

Now, if the registry is on surpriseMode, recipient will not be able to see information related to purchases through queries.

## Q7
By writing User and Item types as generic parameters, we make the concept polymorphic. In this way, we don't need to commit to any particular representation of users or items, but simply assume that some external system will provide opaque identifiers of these types. This enables abstraction and stability: if item details/attributes change over time, the registry logic will still hold. 

SKU code also offers an efficient and handy way to query for items, since we only need to perform one-dimensional lookups instead of checking if all properties like names, descriptions, and prices match up.

# Exercise 2: Extending a familiar concept
## Q1 and Q2
```
concept PasswordAuthentication[User]

purpose limit access to known users

principle
    after a user registers with a username and a password,
    they can authenticate with that same username and password
    and be treated each time as the same user

state
    a set of Users with
        a username String
        a password String

actions
    register (username: String, password: String): (user: User)
        requires: no user exists with this username
        effects:
            creates a new user with this username and password
            returns the new user

    authenticate (username: String, password: String): (user: User)
        requires: exists a user with this username and password
        effects: return that matching user
```

## Q3
**Invariant**: There is at most one user with a given username. This also means we have unique usernames in the User set.

The initial state (base case) is good because by default, we have an empty set of Users. So, the base case is trivial, and the invariant holds for the base case.

During the inductive steps (i.e., during transitions made by the actions), only the **register** action can possibly break the invariant, so we need to guard the **register** action. The precondition "no user exists with this username" ensures that one username will not get registered twice to create two different users. **authenticate** action does not mutate the state, so it will not violate the invariant. Since the initial state is good and no action can violate the invariant (i.e., turning a good state into a bad state), the invariant is preserved.

## Q4
We augment the state and actions to include this functionality.

```
state
    a set of Users with
        a username String
        a password String
        a confirmed Flag   // true if email is confirmed

    a set of Confirmations with
        a user User
        a secretToken String   // secret token generated during registration

actions
    register (username: String, password: String): (user: User, secretToken: String)
        requires: no user exists with this username
        effects:
            create a new user with this username and password
            set confirmed as False for this newly created user
            create a new confirmation for this user with a new secretToken
            return the new user and the secretToken
        
    confirm (username: String, secretToken: String)
        requires:
            exists a user with this username and
            the user is not confirmed and
            exists a confirmation for this user with the matching secretToken
        **effects**
            make the corresponding user's confirmed flag as True
            remove the confirmation for this user

    authenticate (username: String, password: String): (user: User)
        requires:
            exists a user with this username and matching password and has confirmed as True
        effects: return that matching user
```


# Exercise 3: Comparing concepts

```
concept PersonalAccessTokenClassic [User]

purpose
    enable programmatic authentication and access on behalf of a user
    without sharing their password

principle
    a user generates a token with a chosen scope and expiration;
    the system returns the token's secret once;
    the user can authenticate using the token with access limited to the preset scope and expiration;
    the token can be deleted, after which it can not authenticate;

state
    a set of Tokens with
        an owner User
        a secret String
        a scope Scope
        a revoked Flag
        an expiration Time
    
    a set of Users with
        an username String

**actions**
    createToken (owner: User, scope: Scope, expiration: Time): (secret: String):
        requires: exist owner
        effects:
            create a new token with this owner with:
                the scope as the provided or default scope, 
                revoked as false,
                secret as a newly generated secret string,
                expiration as the provided or default time
            return the token's secret once
    
    deleteToken (owner: User, token: Token)
        requires:
            token exists and
            the owner of the token matches the action's owner and
            revoked is false
        **effects** make revoked true
    
    authenticateWithToken (username: String, secret: String) : (user: User)
        requires:
            a token exists with a secret matching the action's input secret, and
            revoked is false for this token, and
            the current timestamp is before expiration for this token
        effects: authentication success
```

It is worth noticing that, according to the GitHub Page, although users are required to enter their username along with the personal access token, the username is not used to authenticate the user. So, in the `authenticateWithToken` action, username is not used in the precondition.

## Difference with PasswordAuthentication
PersonalAccessToken and PasswordAuthentication are both used for authentication, but they serve different use cases and thus have different designs.

For PasswordAuthentication, each user would only have a single password. For PersonalAccessToken, one user can be the owner for multiple tokens at the same time, each with its own scopes and expiration. User can also revoke the tokens individually. This differentiates PersonalAccessToken from PasswordAuthentication.

Consider the following different use cases:
- PasswordAuthentication: log in to the web UI, authenticate as the user for a session to change profile, set up billing information, or interact as a "user" (for example, posting under this user profile).
- PersonalAccessToken: create new tokens with different scopes and expirations for each server, app, bot, API, or deployment. This allows per-integration or per-action permission isolation. PersonalAccessToken would be especially good for dev (and this is why GitHub implements it) because of its environment isolation and separation.

## GitHub page suggestions
- Avoid terms like "personal access tokens are like passwords" (which were mentioned repeatedly throughout the article) to avoid confusion. Focus on describing the concept of personal access tokens.
- Add a comparison section at the bottom to list the different use scenarios for password authentication and personal access token (like what I did above). Focus on why GitHub uses personal access tokens, including elaborating on use cases where "scope" and "expiration" are important.

# Exercise 4: Defining familiar Concepts

## Conference Room Booking

```
concept ConferenceRoomBooking [User, Item]

purpose Let people reserve public conference rooms at specific times to avoid conflict

principle
    an admin creates available slots for rooms for different time
    a user reserve an available slot for a room
    a reserved slot cannot be double-booked
    a user can cancel their reservation
    an admin can deactivate slots

state
    a set of Rooms with
        a capacity Number
        a roomId String    // this roomId is a unique identifier

    a set of Slots with
        a Room
        a Time
        an active Flag
    
    a set of Bookings with
        a User
        a Slot
    
actions
    addRoom (user: User, capacity: Number, roomId: String): (room: Rooms)
        requires: user is admin and no room exists with this roomId
        effects: make room with this capacity and roomId

    createActiveSlot (user: User, room: Room, time: Time): (slot: Slot)
        requires
            user is admin, and
            room exists, and
            no active slot exists with the same room and time
        
        **effects** 
            if a slot already exists with the same room and time, make this slot active;
            otherwise, create a fresh slot for this room and time, and set active as True
        
    deactivateSlot (user: User, slot: Slot)
        requires: user is admin, slot exists, and slot is active
        effects: set slot as not active
    
    reserve (user: User, slot: Slot, capacity: Number): (booking: Booking)
        requires
            slot exists and slot is active
            no booking exists for this slot
            the user requested capacity is less than the capacity of the room for this slot
        
        **effects**
            create a new booking for this user and slot
    
    cancelReserve (user: User, booking: Booking)
        requires: booking exists and has the matching user
        effects: remove booking from the set of Bookings
```

There are two core invariants that needs to be preserved:
1. There is at most one slot per (room, time) pair. The base case is good under this invariant because we initially have an empty set of slots. We guard this invariant by the requires statement for `createActiveSlot`, which explicitly requires "no slot exists with the same room and time."

2. There is at most one booking per slot, avoiding double-booking. The base case is good under this invariant because we initially have an empty set of bookings. We guard this invariant by the requires statement for `reserve`, which checks "no booking exists for this slot."

Since we start with a good base case for both invariants, and no transition/action would turn a good state into a bad state, the invariants are preserved.

It is also worth noticing that admins are special users in charge of maintaining the reservation system. Thus, they are the only users that can call `addRoom`, `createActiveSlot`, and `deactivateSlot` actions.

## Billable Hours Tracking

```
concept BillableHoursTracking [User, Item]

purpose Record billable work sessions per employee per project

principle
    an employee starts a work session by choosing a project and entering a short work description;
    later this employee ends this session;
    if a session exceeds its plannedEnd time (if set) or some default endDuration after start, the system automatically ends the session and marks it as auto-ended

state
    a set of Sessions with
        an employee Employee
        a project Project
        a startTime Time
        an endTime Time (optional until the session has ended)
        a plannedEnd Time (optional)
        a description String
        an autoEnded Flag
    
    a set of Projects with
        a projectId String    // this projectId serves as a unique identifier
    
    a set of Employees with
        an employeeId String    // this employeeId serves as a unique identifier

    a endDuration Duration     // this is the default cutoff time for sessions

actions
    startSession (employee: Employee, project: Project, description: String, startTime: Time, plannedEnd: optional Time): (session: Session)
        requires:
            employee and project exist, and
            no session exists for this employee with the same (project, startTime) pair
        effects:
            create a new session with the given employee, project, startTime, description,
            set endTime for this session as None and autoEnded as false,
            if plannedEnd is provided, set as the given plannedEnd
            
    endSession (employee: Employee, session: Session, endTime: Time):
        requires:
            employee exists, and
            session exists with this employee and has endTime as None, and
            the given endTime is after session's startTime
        effects: set session's endTime as the given endTime
    
    autoEnd (session: Session, currentTime: Time):
        requires
            session exists, and
            session has end as None, and
            currentTime is after session.startTime, and
            (
                (session.plannedEnd is defined and currentTime is later than session.plannedEnd)
                OR
                (currentTime is later than session.startTime + endDuration)
            )
        effects:
            set endTime for the session as currentTime
            set autoEnded for the session as True
```

### Notes
I have implemented several states and actions to handle a case in which someone forgets to end a session.

First, each session has an optional `plannedEnd` time, which the employee can set when they perform `startSession` action. The system will automatically end any session that exceeds its plannedEnd time. If no plannedEnd is set for a session, the system will automatically end the session if it takes longer than the system's `endDuration`. Therefore, no session will last longer than `endDuration`.

Additionally, to differentiate between manually-ended and auto-ended sessions, each session has an `autoEnded` flag, which is true if the session is automatically ended. This information would be helpful when companies are using the tracked record to decide billing.

Companies can calculate the work duration for each session with duration = endTime - startTime.


## URL Shortener

```
concept UrlShortener [User, Item]

purpose Provide short aliases that redirect to long target URLs for convenience

principle
    user chooses a suffix or asking the system to generate one;
    the system ensures the suffix is unique for the domain and returns the short link;
    user navigates to the long target URL's location with the short link;
    user can deactivate the link so it no longer redirects;

state
    a set of Links with
        a domain String
        a suffix String
        a target URL
        an owner User
        an active Flag

**actions**

createCustom (owner: User, domain: String, suffix: String, target: URL) : (link: Link)
    requires:
        no link exists with the same (domain, suffix) pair
    effects:
        create a new link with the given owner, domain, suffix, and target, and set active to true

createAuto (owner: User, domain: Domain, target: URL): (link: Link)
    effects:
        autogenerate a fresh suffix such that no other link exists with the same (domain, suffix) pair,
        create a new link with the given owner, domain, target, and autogenerated suffix, and set active to true

redirect (domain: Domain, suffix: Suffix): (target: URL)
    requires:
        exist a link with matching (domain, suffix) pair and has active set as true
    effects:
        return the target

deactivate (owner: User, link: Link)
    requires:
        link exists, and
        the owner of this link matches the given owner, and
        this link is active
    effects:
        make active false for this link

activate (owner: User, link: Link)
    requires:
        link exists, and
        the owner of this link matches the given owner, and
        this link is inactive
    effects:
        make active true for this link
```

### Notes

The key invariant for the state is there is at most one link per (domain, suffix) pair. The base case (when there is an empty set of Links) is trivial and holds the invariant. We preserve the invariant by:

1. Having a requires statement for `createCustom` that forbids creating shortened URL when the custom (domain, suffix) pair is occupied by another link.

2. Having `createAuto` autogenerate fresh suffix with unoccupied (domain, suffix) pair to prevent collision and violation of the invariant.

Since we start with a good base case, and no transition/action would turn a good state into a bad state, the invariant is preserved.