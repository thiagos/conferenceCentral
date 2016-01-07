Conference Central App
======================

This App consists of a set of backend APIs, implemented using Google App Engine,
to manage participants, conferences and sessions within those conferences.

Each session has its conference entity as its parent. This simplifies the
API 'getConferenceSessions' for instance, as a query by ancestor returns the required info,
 and keeps the data better organized overall.
