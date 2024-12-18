# Logistics API Documentation By Group 1
**Link to REST docs:** https://bpilgroup1-sandbox.mxapps.io/rest-doc/rest

**Link to SOAP docs:** https://bpilgroup1-sandbox.mxapps.io/ws-doc/

This documentation outlines the API endpoints required for communication between the Truck Company, Control Tower, and Distribution Center in a logistics management system. The APIs facilitate the exchange of information about trucks, orders, ETAs, locations, and status updates.

Each API call is detailed below with the required payload, purpose, and structure.

The number of each API call corresponds with the bold red numbers in the Public BPIL diagram.

## API Requests (see API Responses below)

### 1. Send Truck and Order Information

**Sender**: Truck Company

**Receiver**: Control Tower

**Method**: `POST`

**Description**: Sends truck and order details from the Truck Company to the Control Tower.

**Payload if destination is DC**:
**REST Endpoint**: `/rest/truck-collect-orders`

```jsonc
{
    "TCID": "TC10", // Use groupnumber
    "Truck": {
        "LicencePlate": "TC06ABC", // Always start your LicencePlate with TCxx, where x is your group number (with leading 0 if < 10), length = 7 chars, to ensure that every licence plate is unique between the TCs
        "Height": 4,
        "Width": 2.5,
        "Weight": 9000,
        "Location": {
            "Long": 37.7749, // Start location route
            "Lat": -122.4194 // Start location route
        }
        
    },
    "DCID": "DC04",
    "Orders": [
        {
            "OrderID": "AB123",
            "Products": [
                {
                    "SKU": "PT-001", // Refers to "Table with products"
                    "Quantity": 2
                }
            ]
        },
        {
            "OrderID": "AB123",
            "Products": [
                {
                    "SKU": "PT-002", // Refers to "Table with products"
                    "Quantity": 3
                }
            ]
        }
    ]
}
```

**Payload if destination is not DC**:
**REST Endpoint**: `/rest/truck-deliver-to-supermarket`
```jsonc
{
    "TCID": "TC10", // Use groupnumber
    "Truck": {
        "LicencePlate": "TC06ABC", // Always start your LicencePlate with TCxx, where x is your group number (with leading 0 if < 10), length = 7 chars, to ensure that every licence plate is unique between the TCs
        "Height": 4,
        "Width": 2.5,
        "Weight": 9000,
        "Location": {
            "Long": 37.7749, // Start location route
            "Lat": -122.4194 // Start location route
        }
        
    },
    "SPID": "SP01"
}
```

### 2. Share ETA with Truck Company

**REST Endpoint**: `/rest/share-eta`

**Method**: `POST`

**Sender**: Control Tower

**Receiver**: Truck Company

**Description**: Shares the calculated ETA with the Truck Company.

**Payload**:
```jsonc
{
    "LicencePlate": "TC06ABC",
    "ETA": "2024-12-10 16:26:50Z"
}
```

### 3. Share ETA and Order Information with Distribution Center

**REST Endpoint**: `/rest/share-eta`

**Method**: `POST`

**Sender**: Control Tower

**Receiver**: Distribution Center

**Description**: Shares the calculated ETA and order information with the Distribution Center. **Only the orders that are important for that Distribution Center are shared, so not always all the orders in the arriving truck**

**Payload**:
```jsonc
{
    "ControlTowerId": "CT01", // Use group number
    "TruckLicencePlate": "TC06ABC"
    "Orders": [
        {
            "OrderID": "AB123",
            "Products": [
                {
                    "SKU": "PT-001",
                    "Quantity": 2
                    
                }
            ]
        }
    ],
    "ETA": "2024-12-10 16:26:50Z"
}
```

### 4. Request Truck Location Update
This endpoint is abstracted out, because we don't exactly keep track of the location of the truck, but are simulation the ETA. So for keeping track of the ETA, the CT adds some randomness, but doens't send API requests to the TC, because otherwise they have to do the randomness, what doens't add anything to the workflow.

### 5. Send Arrival Notification (15-min ETA) to Truck Company

**SOAP Endpoint**: `/soap/notify-truck-arrival`

**Sender**: Control Tower  

**Receiver**: Truck Company  

**Description**: Notifies the Truck Company that the truck is 15 minutes away from the destination. Uses the license plate as identifier. **For this moment, the ETA is set and doesn't change, this makes it different from the other ETA update request**  

See files starting with "Request 5" for the formatting.

### 6. Send Arrival Notification (15-min ETA) to Distribution Center

**SOAP Endpoint**: `/soap/notify-truck-arrival`

**Sender**: Control Tower

**Receiver**: Distribution Center  

**Description**: Notifies the Distribution Center that the truck is 15 minutes away from the destination. Uses the order id as identifier. **For this moment, the ETA is set and doesn't change, this makes it different from the other ETA update request**  

See files starting with "Request 6" for the formatting.

### 7. Send "Truck Ready for Departure" Notification

**REST Endpoint**: `/rest/notify-truck-ready`

**Method**: `POST`

**Sender**: Distribution Center

**Receiver**: Control Tower

**Description**: Notifies the Control Tower that the truck is loaded and ready for departure.

**Payload**:
```jsonc
{
    "OrderID": "AB123",
    "Details": {
        "LoadedWeight": 8900,
        "LoadedPallets": 4
    }
}
```

### 8. Send "Truck Ready for Departure" Notification

**REST Endpoint**: `/rest/notify-truck-ready`

**Method**: `POST`

**Sender**: Control Tower

**Receiver**: Truck Compnay

**Description**: Notifies the Truck Company that the truck is loaded and ready for departure.

**Payload**:
```jsonc
{
    "TruckLicencePlate": "TC06ABC"
    "Details": {
        "LoadedWeight": 8900,
        "LoadedPallets": 4
    }
}
```

### 9. Request Current ETA from Control Tower

**REST Endpoint**: `/rest/request-current-eta`

**Sender**: Distribution Center

**Receiver**: Control Tower

**Method**: `POST`

**Description**: Sends a request from the Distribution Center to the Control Tower for the current ETA of a specific order.

**Payload**:
```jsonc
{
    "OrderID": "AB123"
}
```

**Response**:
```jsonc
{
    "OrderID": "AB123",
    "ETA": "2024-12-10 16:26:50Z"
}
```

## REST Responses (except for request 8)

Each API endpoind responds with a standard structure that includes a `message`, and, where applicable, additional data to provide context. The HTTP status code of the response provides a general indication of the request's outcome, while the `message` field in the JSON body offer a more detailed explanation of what occurred.

### General Response Structure
```jsonc
{
    "message": "Description of the outcome"
}
```

### Example Succes Response
```jsonc
{
    "message": "Truck and order information received successfully."
}
```

### Example Error Response
```jsonc
{
    "message": "Missing required fields: 'Truck.LicencePlate'."
}
```