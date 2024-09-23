# Aircall.webhook
Push inbound Aircall data to data warehouse to calculate KPIs.
# Aircall Data Warehouse Demo

This repository provides a demonstration of how Aircall's webhook data can be used to calculate important KPIs such as:
- Total Number of Inbound Calls
- Total Number of Missed Calls
- Pickup Rate (%)
- Average Waiting Time

## Webhook Event Flow
This diagram shows the events triggered during an inbound call, which are used to calculate KPIs:
![Call Event Flow](path/to/diagram)

## Sample code for KPI Calculation

The following code outlines how you might process webhook data from Aircall to calculate the required KPIs:

```javascript
const calculateKPIs = (callEvents) => {
  const totalInboundCalls = callEvents.filter(event => event.resource === 'call' && event.event === 'call.created').length;
  const totalMissedCalls = callEvents.filter(event => event.resource === 'call' && event.event === 'call.ended' && !callEvents.some(ansEvent => ansEvent.event === 'call.answered' && ansEvent.data.id === event.data.id)).length;
  const totalAnsweredCalls = callEvents.filter(event => event.resource === 'call' && event.event === 'call.answered').length;
  const pickupRate = (totalAnsweredCalls / totalInboundCalls) * 100;
  const totalWaitingTime = callEvents.filter(event => event.event === 'call.answered').reduce((acc, answeredEvent) => {
    const createdEvent = callEvents.find(event => event.event === 'call.created' && event.data.id === answeredEvent.data.id);
    const waitingTime = answeredEvent.timestamp - createdEvent.timestamp;
    return acc + waitingTime;
  }, 0);
  const avgWaitingTime = totalAnsweredCalls > 0 ? totalWaitingTime / totalAnsweredCalls : 0;

  return {
    totalInboundCalls,
    totalMissedCalls,
    pickupRate,
    avgWaitingTime
  };
};
