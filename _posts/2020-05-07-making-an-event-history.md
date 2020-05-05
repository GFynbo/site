---
layout: post
title:  "Making an Event History for Properties"
date:   2020-05-07 12:00:00 -0700
categories: [javascript]
---

The first thing we need to do when creating the event history is decide how the events will be associated and what information they need to include to be useful.

In our case, we're looking to attach related events by `propertyId` and at the least have a couple additional properties. We're looking to have a `cost` assocaited with it, a `datetime` of the event (this is different than the creation time of the event object) and a `notes` field to allow additional information.


```javascript
const event = {
    eventId: String,
    eventCreationTime: moment(),
    propertyId: String,
    cost: Number,
    notes: String
};
```

Article will be finished shortly.
