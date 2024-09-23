This repository showcases how to use the Aircall API to fetch inbound call data and calculate relevant KPIs using JavaScript and Axios.

## Overview

This project demonstrates how to:
1. Fetch inbound call data from the Aircall API using the `/v1/calls` endpoint.
2. Filter and calculate KPIs such as:
   - Total number of inbound calls
   - Pickup rate (% of calls answered)

The code is written in JavaScript using the **Axios** library for making API requests and handles filtering for specific call data (e.g., inbound vs. outbound).

## API Endpoint

The main API endpoint used in this demo is:

**GET** `https://api.aircall.io/v1/calls`

- **Query Parameters**:
  - `direction=inbound`: Fetches only inbound calls.
  - `per_page=100`: (Optional) Specifies the number of records to fetch per page (up to 100).

### Example Request:

```bash
GET https://api.aircall.io/v1/calls?direction=inbound&per_page=100
