Conference Central App
======================

This App consists of a set of backend APIs, implemented using Google App Engine,
to manage participants, conferences and sessions within those conferences.

Each session has its conference entity as its parent. This simplifies the
API 'getConferenceSessions' for instance, as a query by ancestor returns the required info,
 and keeps the data better organized overall.

The requirements on how to query conferences were open enough so that a generic
'ConferenceQueryForm' message, with (field, operator, value) keys was enough. However, 
the requirements on Sessions queries are slightly different, since specific APIs like 
'getSessionsBySpeaker' and 'getConferenceSessionsByType' were required.

Based on that, even though a generic 'SessionQueryForm', with same fields as its 
Conference correspondent, would work, the user experience wouldn't be the best:
in API 'getSessionsBySpeaker', the user would need to fill the field 'field' with value
'speaker', and field 'value' with the speaker name (confusing, right... :) )

To avoid that, I prefered to create specific inbound messages for each specific case, i.e.,
classes 'SessionSpeakerQueryForm' and 'SessionTypeQueryForm' were created, with just the 
specific field used by filter as parameter.