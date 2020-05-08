---
layout: post
title:  "Making an Event History"
date:   2020-05-07 12:00:00 -0700
categories: [javascript]
---

## Making an Event History

A quick three step recap/guide/whatever to building an event history for whatever web application you're developing.

## Inception

The first thing we need to do when creating the event history is decide how the events will be associated and what information they need to include to be useful.

In our case, we're looking to attach related events by `propertyId` and at the least have a couple additional properties. We're looking to have a `cost` assocaited with it, a `datetime` of the event (this is different than the creation time of the event object) and a `notes` field to allow additional information.


```javascript
const propertyEvent = {
    eventId: String,
    eventCreationTime: moment(),
    eventDate: Date,
    propertyId: String,
    cost: Number,
    notes: String
};
```

For each time that we want to add a `propertyEvent` to a property we need to include this `propertyId` to associate this event with the corresponding property thus allowing us to grab this via the relationship when loading a particular property.

An important thought now that we've defined this `propertyEvent` object is, when and how should we retrieve this event stream from the database?

There are a couple methods, especially since in our context we will be using Redux. We could pull all the history for every property on the initial `getProperties` and store it in the global store so that all component have access where necessary without any additional calls. However, there's other methods that might be easier on both the client and the server which is what we will move forward with in our design. In particular, I will move forward with a lazy-load style approach. In this approach, when we load the view of a particular property we will then send a request to the backend to retrieve all `propertyEvent`s associated with that `propertyId` and this will reduce our load directly to each property and maintain a cleaner state.

```javascript
componentDidMount() {
  fetch(url, {
    method: 'POST', // *GET, POST, PUT, DELETE, etc.
    mode: 'same-origin', // no-cors, *cors, same-origin
    cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
    credentials: 'same-origin', // include, *same-origin, omit
    headers: {
      'Content-Type': 'application/json'
      // 'Content-Type': 'application/x-www-form-urlencoded',
    },
    redirect: 'follow', // manual, *follow, error
    referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
    body: JSON.stringify(data) // body data type must match "Content-Type" header
  })
  .then(data => {
      MOVE_TO_STORE(data);
  })
  .catch(err => {
      console.error(err);
  });
}
```

Now that we have our object, and retrieval methods in place we need to design our components for displaying, editing, and adding new events to our event history for this property.

### Displaying

We can display this event history simply using a `<Card>` component in [Material-UI](https://material-ui.com) for basic information. We will also want to include a couple of buttons on this card under the actions section to allow edit and delete of an event.

### Editing

We will present essentially the same form as adding with the information auto-filled and an update button. This will follow the same process as adding except with an update on the backend.

### Adding

Another method we can use here is simply create a modal with a simple form that sends to the backend with the `notes`, `propertyId`, `cost`, and `eventDate`. The backend will assign the creation time and associated data. We will also confirm that the property we're assigning belongs to the authenticated user.

## Implementation (or an attempt at it)

Now that we've designed our system we want to implement it or attempt to. As I said, easier said than done.

### MongoDB model

We essentially wrote exactly what we needed for the MongoDB model above, I'll copy it below, but we will name this event `PropertyEvent`.

```javascript
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const PropertyEventSchema = new Schema({
  eventId: {
    type: String,
    required: true
  },
  eventCreationTime: {
    type: Date,
    required: true
  },
  eventDate: {
    type: Date,
    required: true
  },
  propertyId: {
    type: String,
    required: true
  },
  cost: {
    type: Number,
    required: true
  },
  notes: {
    type: String,
  }
});
module.exports = PropertyEvent = mongoose.model("propertyEvent", PropertyEventSchema);
```

### API endpoints

Now we need to build these endpoints to allow users to create, update, and delete event history for this property. As I already have defined routing for a number of endpoints for properties, we will just build that into the existing routing. If you're looking to setup routing for the first time I'd follow this guide: [Express Routing](https://expressjs.com/en/guide/routing.html). We will also need a endpoint that retrieves all the events for a given property. We will build this first.

```javascript

```

### Modify view component

### Event components

#### Display card

#### Add form

#### Edit form

## Conclusion

A learning experience. TODO.

Article will be finished shortly.
