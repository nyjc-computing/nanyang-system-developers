# Event-Driven Architecture

## Introduction

In a more "traditional" application, the app might:

1. call some functions, or make an API request
2. check the data for changes
3. If there are changes to be handled, handle the changes
4. If there are no changes, wait/sleep for a suitable interval
5. Repeat steps 1-4 in an infinite loop, or until an exit signal is received.

This pattern is simple, but suffers the following limitations:

- **Resource-wasteful**: need to call functions/make API requests even when no data has changed
- **Difficult to extend**: each new service or data source to be checked requires a new API call

## Mitigation

Event-driven architecture (EDA) is a software design pattern centred around **events**. Events are packets of data containing:

- a text label
- optionally, additional data about the event

Changes to data are signalled using events.

An app **subscribes** to events, usually by attaching a **handler** i.e. a function/method that is called when an event is received. Such apps are called **subscribers**.

An app can also **emit** events, whenever it has caused or triggered a data change. Such apps are called **publishers** (alt. **producers**).

An app can be both a subscriber and a publisher.

The exchange of events between subscribers and publishers is mediated by an **event broker** (alt. message brokers, message queue).

## Benefits of Event-Driven Architecture

- **Scalability**: Decoupling components allows systems to scale independently.
- **Flexibility**: New features can be added without disrupting existing components.
- **Resilience**: Failures in one component do not necessarily affect others.

## Example: User Registration Workflow

1. **Event Producer**: A web application generates a `"user.registered"` event.
2. **Event Broker**: The event is sent to a message queue.
3. **Event Consumers**:
   - A notification service sends a welcome email.
   - An analytics service logs the registration event.

## Further Reading

- [NYSD Project Discussion #001: Event-Driven Architecture](https://docs.google.com/presentation/d/e/2PACX-1vRls2WzEPjbnreRgUaHEM46IJ2iyznz7xWpW13XaZU_mJ-CLEr7A88CWl9N3ghRN19lyx0-pAp80Lf7/pub?start=false&loop=false&delayms=60000)
