# Aircall.webhook
Push inbound Aircall data to data warehouse to calculate KPIs.
# Aircall Data Warehouse Demo

This repository provides a demonstration of how Aircall's webhook data can be used to calculate important KPIs such as:
- Total Number of Inbound Calls
- Total Number of Missed Calls
- Pickup Rate (%)
- Average Waiting Time

To successfully demonstrate how API endpoints are used to calculate KPIs based on Aircall data, you need to understand which endpoints provide access to the relevant data (e.g., call logs, call events). For each KPI, you’ll be interacting with specific API endpoints that allow you to fetch data from Aircall.

For this scenario, we are working with **inbound calls**, so the primary API endpoints will come from **Aircall's call-related APIs**. 

#### 1. **Total Number of Inbound Calls**
   - **API Endpoint**: `GET https://api.aircall.io/v1/calls`
   - **Purpose**: This endpoint retrieves the call logs. You can filter the response to only include inbound calls.
   - **Query Parameter Example**:
     ```bash
     GET https://api.aircall.io/v1/calls?direction=inbound
     ```

#### 2. **Total Number of Missed Calls**
   - **API Endpoint**: `GET https://api.aircall.io/v1/calls`
   - **Purpose**: Use the same endpoint to retrieve call logs and filter for inbound calls with a **missed status**.
   - **Query Parameter Example**:
     ```bash
     GET https://api.aircall.io/v1/calls?direction=inbound&status=missed
     ```

#### 3. **Pickup Rate (%)**
   - **API Endpoint**: `GET https://api.aircall.io/v1/calls`
   - **Purpose**: Fetch the total number of inbound calls and the number of answered calls to calculate the pickup rate.
   - **Query Parameter Example**:
     ```bash
     GET https://api.aircall.io/v1/calls?direction=inbound&status=answered
     ```

   **Pickup Rate Calculation**:
   You’ll need to query for both **answered** and **total inbound calls**, then calculate the rate:
   ```js
   pickupRate = (totalAnsweredCalls / totalInboundCalls) * 100;
   ```

#### 4. **Average Waiting Time**
   - **API Endpoint**: `GET https://api.aircall.io/v1/calls`
   - **Purpose**: To calculate waiting time, you will need to retrieve call logs that include both **call.created** and **call.answered** timestamps. You can calculate the difference between the two for each answered call.
   - **Query Parameter Example**:
     ```bash
     GET https://api.aircall.io/v1/calls?direction=inbound&status=answered
     ```

   **Calculation**:
   Using the API response, you can extract the `started_at` and `answered_at` fields:
   ```js
   const waitingTime = answeredEvent.answered_at - createdEvent.started_at;
   ```

### **How to Use These Endpoints**:
Each of these endpoints allows you to fetch call data from Aircall. Once you have the data, you can process it in your code to calculate the KPIs.

For example, to get **Total Number of Inbound Calls**, you would:

1. Make an HTTP request to the endpoint:
   ```bash
   GET https://api.aircall.io/v1/calls?direction=inbound
   ```
   
2. The response will include a list of all inbound calls. You can use the length of this list to determine the total number of inbound calls.

### Example API Response for Call Logs:
Here's a simplified version of what the response might look like when calling `GET /calls`:
```json
{
  "meta": {
    "count": 10
  },
  "calls": [
    {
      "id": 12345,
      "direction": "inbound",
      "started_at": "2024-09-23T10:00:00Z",
      "answered_at": "2024-09-23T10:00:10Z",
      "status": "answered"
    },
    {
      "id": 12346,
      "direction": "inbound",
      "started_at": "2024-09-23T10:05:00Z",
      "status": "missed"
    }
  ]
}
```
From this response, you could calculate:
- **Total Inbound Calls**: Count the number of calls where `"direction": "inbound"`.
- **Total Missed Calls**: Filter the calls where `"status": "missed"`.
- **Pickup Rate**: Use the `"answered"` calls and divide by the total inbound calls.
- **Average Waiting Time**: Subtract `started_at` from `answered_at` for each answered call.

### Conclusion:
To sum up:
- **Total Inbound Calls**: `GET /calls?direction=inbound`
- **Total Missed Calls**: `GET /calls?direction=inbound&status=missed`
- **Pickup Rate**: `GET /calls?direction=inbound&status=answered`
- **Average Waiting Time**: Calculated from the timestamps in the response (`started_at` and `answered_at`).

These endpoints give you the raw data, and you can process the responses in your code to calculate the required KPIs.

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
