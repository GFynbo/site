---
layout: post
title:  "Making an Event History (the backend)"
date:   2020-05-10 12:00:00 -0700
categories: [javascript]
---

## Making an Event History (the backend)

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

Now we need to build these endpoints to allow users to create, update, and delete event history for this property. As I already have defined routing for a number of endpoints for properties, we will just build that into the existing routing. If you're looking to setup routing for the first time I'd follow this guide: [Express Routing](https://expressjs.com/en/guide/routing.html). We will also need a endpoint that retrieves all the events for a given property. The following requests need to be filled out for a complete end-to-end cycle for events. The `authenticate` function is a custom function I have to validate the JWT from the user before allowing the API to continue.

```javascript
router.get('/events/:propertyId', authenticate, (req, res, next) => {
    // TODO
});

router.post('/events/create', authenticate, (req, res, next) => {
    // TODO
});

router.patch('/events/update', authenticate, (req, res, next) => {
    // TODO
});

router.delete('/events/delete', authenticate, (req, res, next) => {
    // TODO
});
```

We also need to validate our data coming in via the form to ensure that we're not making an objects with bad or blank information. Using the `validator` library makes this a lot easier. With and example like below.

```javascript
const Validator = require("validator");
const isEmpty = require("is-empty");
module.exports = function validateEventInput(data) {
    let errors = {};

    data.eventCreationTime = !isEmpty(data.eventCreationTime) ? data.eventCreationTime : "";
    data.eventDate = !isEmpty(data.eventDate) ? data.eventDate : "";
    data.propertyId = !isEmpty(data.propertyId) ? data.propertyId : "";
    data.cost = !isEmpty(data.cost) ? data.cost : "";
    data.notes = !isEmpty(data.notes) ? data.notes : "";

    if (Validator.isEmpty(data.eventCreationTime)) {
        errors.eventCreationTime = "Event creation time is required";
    }
    if (Validator.isEmpty(data.eventDate)) {
        errors.eventDate = "Event date field is required";
    }
    if (Validator.isEmpty(data.propertyId)) {
        errors.propertyId = "Property id is required";
    }
    if (Validator.isEmpty(data.cost)) {
        errors.cost = "Cost field is required";
    }
    if (Validator.isEmpty(data.notes)) {
        errors.notes = "Notes field is required";
    }
    return {
        errors,
        isValid: isEmpty(errors)
    };
};
```

For the `GET` request of all events for a particular property we want to grab them by `propertyId` as follows. You begin by checking that this property exists and that it is owned by the same user as the one making the request and then you retrieve all events with that ID.

```javascript
router.get('/events/:propertyId', authenticate, (req, res, next) => {
    const propertyId = req.params.propertyId;

    Property.findOne({_id: propertyId}, (err, property) => {
        if (err) next(err);
        else if (property) {
            if (property.userId !== req.user.id) {
                return res.sendStatus(403);
            } else {
                PropertyEvent.find({propertyId: propertyId}, (err, events) => {
                    if (err) {
                        next(err);
                        return res.sendStatus(403);
                    } else {
                        return res.json({events: events});
                    }
                });
            }
        } else {
            return res.sendStatus(403);
        }
    });
});
```

Now for adding an event to a property you want to validate using same ideas as above, but check that the information being sent is also valid with the validator above through the `POST` request.

```javascript
router.post('/events/:propertyId/create', authenticate, (req, res, next) => {
    const event = req.body;
    const propertyId = req.params.propertyId;
    const { errors, isValid } = validateEventInput(event);
    // Check validation
    if (!isValid) {
        return res.status(400).json(errors);
    }

    event.eventCreationTime = Date.now();
    event.eventId = uuid.v4();
    
    const newEvent = new PropertyEvent(event);

    Property.findOne({_id: propertyId}, (err, property) => {
        if (err) next(err);
        else if (property) {
            if (property.userId !== req.user.id) {
                return res.sendStatus(403);
            } else {
                newEvent.save(err => {
                    if (err) next(err);
                    else return res.json({ newEvent, msg: 'Event successfully saved' });
                });
            }
        } else {
            return res.sendStatus(403);
        }
    });
});
```

Now for updating events we want to check this `PATCH` request to do the same thing as the create except this time we'll be using the mongoose `findOneAndUpdate()` method.

```javascript
router.patch('/events/:propertyId/update', authenticate, (req, res, next) => {
    const event = req.body;
    const propertyId = req.params.propertyId;
    const { errors, isValid } = validateEventInput(event);
    // Check validation
    if (!isValid && event.eventId) {
        return res.status(400).json(errors);
    }
    
    Property.findOne({_id: propertyId}, (err, property) => {
        if (err) next(err);
        else if (property) {
            if (property.userId !== req.user.id) {
                return res.sendStatus(403);
            } else {
                PropertyEvent.updateOne({eventId: event.eventId}, {$set: event}, err => {
                    if (err) {
                        next(err);
                        return res.sendStatus(403);
                    } else {
                        return res.sendStatus(200);
                    }
                });
            }
        } else {
            return res.sendStatus(403);
        }
    });
});
```

Finally, we want to be able to delete events of a particular ID. You should check to ensure that this user is allowed to do this for a particular property and that only one can be deleted at any time.

```javascript
router.delete('/events/:propertyId/delete', authenticate, (req, res, next) => {
    const event = req.body;
    const propertyId = req.params.propertyId;
    const { errors, isValid } = validateEventInput(event);
    // Check validation
    if (!isValid) {
        return res.status(400).json(errors);
    }
    
    Property.findOne({_id: propertyId}, (err, property) => {
        if (err) next(err);
        else if (property) {
            if (property.userId !== req.user.id) {
                return res.sendStatus(403);
            } else {
                PropertyEvent.findOneAndDelete({eventId: event.eventId}, err => {
                    if (err) {
                        next(err);
                        return res.sendStatus(403);
                    } else {
                        return res.sendStatus(200);
                    }
                });
            }
        } else {
            return res.sendStatus(403);
        }
    });
});
```

## Conclusion

This wraps up how to build the backend for event history we can move on to the much more interesting frontend. I actually prefer backend coding, but nothing gives me the satisfaction of seeing my work being used like some mediocre frontend components. I hope this helps you build out whatever feature you're looking to build on your site or clear up any confusion.
