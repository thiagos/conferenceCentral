#Conference Central App


This App consists of a set of backend APIs, implemented using Google App Engine,
to manage participants, conferences and sessions within those conferences.


##Task 1: Add Sessions to a Conference

###Sessions ancestors

Each session has its conference entity as its parent. This simplifies the
API *getConferenceSessions* for instance, as a query by ancestor returns the required info,
and keeps the data better organized overall. 

###Session speaker

For simplicity, for now, the speaker will consist of a string field on the Session entity.

###Sessions Messages Implementation

The requirements on how to query conferences were open enough so that a generic
'ConferenceQueryForm' message, with (field, operator, value) keys was enough. However, 
the requirements on Sessions queries are slightly different, since specific APIs like 
*getSessionsBySpeaker* and *getConferenceSessionsByType* were required.

Based on that, even though a generic 'SessionQueryForm', with same fields as its 
Conference correspondent would work, the user experience wouldn't be the best:
in API *getSessionsBySpeaker*, the user would need to fill the field 'field' with value
'speaker', and field 'value' with the speaker name (confusing, right... :) )

To avoid that, I prefered to create specific inbound messages for each case, i.e.,
classes 'SessionSpeakerQueryForm' and 'SessionTypeQueryForm' were created, with just the 
specific field used by each filter as parameter.

##Task 2: Add Sessions to User Wishlist

Similar to conferences to attend, the session wishilist keys will be stored as a list 
into the user profile. 

The information retrieved by API *getSessionsInWishlist* is already present in API *getProfile*,
but a separate Message class with just the sessionsWishlist was created for this purpose anyways.
 
##Task 3: Work on indexes and queries


For all the APIs required up to this point, no query by more than one property was required.
as indexes on single properties are created by default, no manual index creation on the Sessions
entity was needed so far.

To use new indexes, 2 new APIs were created:
- *getQuickWorkshops*: return all workshops only with duration at most 2h.
- *getSessionsBySpeakerAndDate*: returns all sessions from a specific speaker, on a specific date.

For that, 2 new composite indexes were created in index.yaml file:
```
- kind: Session
 properties:
 - name: speaker
 - name: date

- kind: Session
 properties:
 - name: typeOfSession
 - name: duration
```

###The problematic query

The problem with finding all non-workshop sessions before 7PM is that is uses inequality filters
in more than one property.

One possible solution in our scenario is based on the fact that in our system design, the type of sessions 
are enumerated, so we can turn the inequality on the typeOfSession into a 'member of' filter, where 
instead of looking for sessionType != 'WORKSHOP', we can look for sessionType in ['LECTURE', 'KEYNOTE'].

This enumeration shall be clear for the endUser, and new kinds of sessions would require a new system
deployment to be supported.

This query was implemented in API *getEarlyNonWorkshops*.

##Task 4: Add a task

In the event of a session creation, a task will query all sessions on that conference, and if there are 
2 or more sessions from this current speaker, he will be featured via the *getFeaturedSpeaker* API.

The memcache was used for that, populating a single key named 'FEATURED_SPEAKER'. 

No management for cleanup of that key was implemented, but it could be done using a cron task, which 
would periodically remove the memcache entry in case all sessions there are less than 2 sessions from that 
speaker still to happen.