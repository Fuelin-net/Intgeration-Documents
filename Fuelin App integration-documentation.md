# Fuelin App Integration Guide

## Overview
This document outlines the integration process between any Android application and the Fuelin Flutter application, specifically focusing on NFC tag scanning, order processing, authentication workflows, and other core functionality.

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-03-08 | Initial integration documentation |

## Integration Scenarios

### 1. Authentication
- Your app calls Fuelin login screen
- User enters phone and password
- Fuelin returns authentication status

### 2. Shift Management
- After login, your app checks if shift is open
- If no shift is open, your app must call start_shift
- Fuelin starts shift and confirms

### 3. NFC Tag Processing
- Your app reads NFC tag and sends the serial number to Fuelin
- Fuelin processes the tag and returns order details
- Your app displays or processes the order information

### 4. Order Quantity Update
- Your app sends order_id and new quantity to Fuelin
- Fuelin updates the order and confirms success

### 5. Order Details
- Your app can request specific order details by sending order_id to transactions

## Technical Implementation

### Intent Action Definitions

| Function | Intent Action | Description |
|----------|---------------|-------------|
| Login | `com.fuelin.attendant.LOGIN` | Authenticate user |
| Start Shift | `com.fuelin.attendant.START_SHIFT` | Open a new shift |
| NFC Order | `com.fuelin.attendant.NFC_ORDER` | Process NFC tag and return order details |
| Update Order | `com.fuelin.attendant.UPDATE_ORDER` | Update order quantity |
| Transaction Details | `com.fuelin.attendant.TRANSACTIONS` | Get specific order details |

### 1. Authentication

#### Parameters to Send to Fuelin

| Parameter Name | Type | Required | Description |
|---------------|------|----------|-------------|
| phone | String | Yes | User's phone number |
| password | String | Yes | User's password |

#### Example Code (Java)

```java
// Initialize the intent with proper action
Intent intent = new Intent("com.fuelin.attendant.LOGIN");
intent.addCategory(Intent.CATEGORY_DEFAULT);

// Add login credentials
intent.putExtra("phone", phoneNumber);
intent.putExtra("password", password);

// Start Fuelin app and wait for result
startActivityForResult(intent, REQUEST_LOGIN);
```

#### Response from Fuelin

Fuelin will return login status and user information.

#### Example Code for Handling Login Results

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    
    if (requestCode == REQUEST_LOGIN) {
        if (resultCode == RESULT_OK) {
            // Process successful login
            String jsonResponse = data.getStringExtra("response_data");
            
            try {
                JSONObject response = new JSONObject(jsonResponse);
                String message = response.getString("message");
                
                if (message.equals("Success")) {
                    // User is authenticated
                    // Check if shift needs to be opened
                    boolean isShiftOpen = response.getBoolean("is_shift_open");
                    
                    if (!isShiftOpen) {
                        // Need to open a shift
                        startShift();
                    } else {
                        // Shift already open, can proceed with operations
                    }
                } else {
                    // Handle login failure
                    String errorMessage = response.optString("error", "Authentication failed");
                    // Show error to user
                }
            } catch (JSONException e) {
                // Handle JSON parsing error
            }
        } else if (resultCode == RESULT_CANCELED) {
            // User canceled the login operation
        }
    }
}
```

### 2. Shift Management

#### Parameters to Send to Fuelin

| Parameter Name | Type | Required | Description |
|---------------|------|----------|-------------|
| (No parameters required) |  |  | Fuelin will use the currently logged-in user |

#### Example Code (Java)

```java
// Initialize the intent with proper action
Intent intent = new Intent("com.fuelin.attendant.START_SHIFT");
intent.addCategory(Intent.CATEGORY_DEFAULT);

// Start Fuelin app and wait for result
startActivityForResult(intent, REQUEST_START_SHIFT);
```

#### Response from Fuelin

Fuelin will return a JSON response confirming the shift has been started:

```json
{
    "message": "Success",
    "data": {
        "shift_id": 123,
        "start_time": "2025-03-08 09:15:22"
    }
}
```

### 3. NFC Order Processing

#### Parameters to Send to Fuelin

| Parameter Name | Type | Required | Description |
|---------------|------|----------|-------------|
| nfc_tag | String | Yes | Serial number from the NFC tag |

#### Example Code (Java)

```java
// Initialize the intent with proper action
Intent intent = new Intent("com.fuelin.attendant.NFC_ORDER");
intent.addCategory(Intent.CATEGORY_DEFAULT);

// Add required NFC tag data
intent.putExtra("nfc_tag", nfcTagSerialNumber);

// Start Fuelin app and wait for result
startActivityForResult(intent, REQUEST_NFC_ORDER);
```

#### Response from Fuelin

Fuelin will return a JSON response in the following format:

```json
{
    "message": "Success",
    "data": {
        "order_id": 5760,
        "order_status": "Completed",
        "order_quantity": "10",
        "order_fuel_type": "Diesel",
        "order_total_price": "60",
        "consumption_rate": "0",
        "plate": "2781 A D S",
        "speedometer": "3401691",
        "created_at": "2024-09-26 02:16:29",
        "relationShips": {
            "merchant": {
                "id": 211,
                "name_en": "Night",
                "name_ar": "ليلي",
                "registration_id": "11",
                "tax_id": "4345"
            },
            "driver": {
                "id": 668,
                "name": "Mohamed Ahmed Elsaid"
            },
            "company": {
                "id": 37,
                "name_en": "Rady Trance",
                "name_ar": "راضي ترانس"
            }
        }
    }
}
```

#### Example Code for Handling NFC Order Results

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    
    if (requestCode == REQUEST_NFC_ORDER) {
        if (resultCode == RESULT_OK) {
            // Process successful result
            String jsonResponse = data.getStringExtra("response_data");
            
            try {
                JSONObject response = new JSONObject(jsonResponse);
                String message = response.getString("message");
                
                if (message.equals("Success")) {
                    JSONObject orderData = response.getJSONObject("data");
                    int orderId = orderData.getInt("order_id");
                    String orderStatus = orderData.getString("order_status");
                    String quantity = orderData.getString("order_quantity");
                    String fuelType = orderData.getString("order_fuel_type");
                    String totalPrice = orderData.getString("order_total_price");
                    // Process the rest of the order data
                    
                    // Access relationship data
                    JSONObject relationships = orderData.getJSONObject("relationShips");
                    JSONObject merchant = relationships.getJSONObject("merchant");
                    JSONObject driver = relationships.getJSONObject("driver");
                    JSONObject company = relationships.getJSONObject("company");
                    
                    // Process merchant, driver, company data
                } else {
                    // Handle error message
                }
            } catch (JSONException e) {
                // Handle JSON parsing error
            }
        } else if (resultCode == RESULT_CANCELED) {
            // User or system canceled the operation
        }
    }
}
```

### 4. Update Order

#### Parameters to Send to Fuelin

| Parameter Name | Type | Required | Description |
|---------------|------|----------|-------------|
| order_id | Integer | Yes | ID of the order to update |
| quantity | String | Yes | New quantity for the order |

#### Example Code (Java)

```java
// Initialize the intent with proper action
Intent intent = new Intent("com.fuelin.attendant.UPDATE_ORDER");
intent.addCategory(Intent.CATEGORY_DEFAULT);

// Add required update data
intent.putExtra("order_id", orderId);
intent.putExtra("quantity", newQuantity);

// Start Fuelin app and wait for result
startActivityForResult(intent, REQUEST_UPDATE_ORDER);
```

#### Response from Fuelin

Fuelin will return a JSON response confirming the update:

```json
{
    "message": "Success",
    "data": null
}
```

### 5. Transaction Details

#### Parameters to Send to Fuelin

| Parameter Name | Type | Required | Description |
|---------------|------|----------|-------------|
| order_id | Integer | Yes | ID of the order to retrieve |

#### Example Code (Java)

```java
// Initialize the intent with proper action
Intent intent = new Intent("com.fuelin.attendant.TRANSACTIONS");
intent.addCategory(Intent.CATEGORY_DEFAULT);

// Add order ID
intent.putExtra("order_id", orderId);

// Start Fuelin app and wait for result
startActivityForResult(intent, REQUEST_TRANSACTION_DETAILS);
```

## Integration Flow

### Complete Application Flow

1. Your app initiates login to Fuelin
2. User authenticates with phone and password
3. Your app checks if shift is open
4. If no shift is open, your app calls start_shift
5. Your app reads NFC tag from card/device
6. Your app sends NFC tag serial number to Fuelin
7. Fuelin processes the tag and returns order details
8. Your app displays the order information
9. If needed, your app can update order quantity
10. Your app can also request specific order details using the transactions call

## Error Handling

### Response Status Messages

| Message | Description | Action Required |
|---------|-------------|----------------|
| "Success" | Operation completed successfully | Process returned data if applicable |
| "Invalid NFC Tag" | The NFC tag is not recognized | Ask user to rescan or use a different tag |
| "Order Not Found" | The specified order ID doesn't exist | Verify order ID or create a new order |
| "Authentication Failed" | Login credentials are incorrect | Ask user to retry with correct credentials |
| "Shift Already Open" | Attempting to open a shift while one is already active | Continue with normal operation |
| "Network Error" | Communication with server failed | Check network connection and retry |

## Testing the Integration

1. **Test Mode**: Add parameter `test_mode` with value `true` to enter test mode
2. **Test NFC Tags**: The following test tag numbers can be used:
   - "NFC123456" - Returns a completed diesel order
   - "NFC789012" - Returns a pending gasoline order
   - "NFC000000" - Returns an error for testing error handling

## App Requirements

- Minimum Android API Level: 21 (Android 5.0)
- Required Permissions:
  - `android.permission.NFC` - For NFC tag reading
  - `android.permission.INTERNET` - For network communication
- Fuelin App Version: 2.5.0 or higher required for this integration

## Troubleshooting

### Common Issues

1. **Cannot read NFC tag**
   - Ensure NFC is enabled on the device
   - Check if the tag is properly placed on the reader
   - Verify the device supports NFC

2. **Login fails**
   - Check network connectivity
   - Verify credentials are correct
   - Ensure Fuelin servers are operational

3. **No response from Fuelin app**
   - Verify Fuelin app is installed and up-to-date
   - Check intent action names are correct
   - Ensure all required parameters are included








## Appendix: Sample Integration Code

### Java - Complete Integration Sample

```java
public class FuelinIntegrationManager {
    private static final int REQUEST_LOGIN = 1001;
    private static final int REQUEST_START_SHIFT = 1002;
    private static final int REQUEST_NFC_ORDER = 1003;
    private static final int REQUEST_UPDATE_ORDER = 1004;
    private static final int REQUEST_TRANSACTION_DETAILS = 1005;
    
    private final Activity activity;
    private boolean isLoggedIn = false;
    private boolean isShiftOpen = false;
    
    public FuelinIntegrationManager(Activity activity) {
        this.activity = activity;
    }
    
    public void initiateLogin(String phone, String password) {
        Intent intent = new Intent("com.fuelin.attendant.LOGIN");
        intent.addCategory(Intent.CATEGORY_DEFAULT);
        intent.putExtra("phone", phone);
        intent.putExtra("password", password);
        activity.startActivityForResult(intent, REQUEST_LOGIN);
    }
    
    public void startShift() {
        if (!isLoggedIn) {
            throw new IllegalStateException("Must be logged in before starting shift");
        }
        
        Intent intent = new Intent("com.fuelin.attendant.START_SHIFT");
        intent.addCategory(Intent.CATEGORY_DEFAULT);
        activity.startActivityForResult(intent, REQUEST_START_SHIFT);
    }
    
    public void processNfcTag(String nfcTagSerial) {
        if (!isLoggedIn || !isShiftOpen) {
            throw new IllegalStateException("Must be logged in and have an open shift");
        }
        
        Intent intent = new Intent("com.fuelin.attendant.NFC_ORDER");
        intent.addCategory(Intent.CATEGORY_DEFAULT);
        intent.putExtra("nfc_tag", nfcTagSerial);
        activity.startActivityForResult(intent, REQUEST_NFC_ORDER);
    }
    
    public void updateOrderQuantity(int orderId, String quantity) {
        if (!isLoggedIn || !isShiftOpen) {
            throw new IllegalStateException("Must be logged in and have an open shift");
        }
        
        Intent intent = new Intent("com.fuelin.attendant.UPDATE_ORDER");
        intent.addCategory(Intent.CATEGORY_DEFAULT);
        intent.putExtra("order_id", orderId);
        intent.putExtra("quantity", quantity);
        activity.startActivityForResult(intent, REQUEST_UPDATE_ORDER);
    }
    
    public void getTransactionDetails(int orderId) {
        if (!isLoggedIn) {
            throw new IllegalStateException("Must be logged in to access transactions");
        }
        
        Intent intent = new Intent("com.fuelin.attendant.TRANSACTIONS");
        intent.addCategory(Intent.CATEGORY_DEFAULT);
        intent.putExtra("order_id", orderId);
        activity.startActivityForResult(intent, REQUEST_TRANSACTION_DETAILS);
    }
    
    public void handleActivityResult(int requestCode, int resultCode, Intent data) {
        if (resultCode != Activity.RESULT_OK) {
            // Handle cancellation or error
            return;
        }
        
        switch (requestCode) {
            case REQUEST_LOGIN:
                // Process login result
                isLoggedIn = true;
                // Check if shift needs to be started
                break;
                
            case REQUEST_START_SHIFT:
                // Process shift start result
                isShiftOpen = true;
                break;
                
            case REQUEST_NFC_ORDER:
                // Process NFC order result
                String jsonResponse = data.getStringExtra("response_data");
                // Parse and handle the JSON response
                break;
                
            case REQUEST_UPDATE_ORDER:
                // Process order update result
                break;
                
            case REQUEST_TRANSACTION_DETAILS:
                // Process transaction details result
                break;
        }
    }
}
```

### Kotlin - Complete Integration Sample

```kotlin
class FuelinIntegrationManager(private val activity: Activity) {
    companion object {
        private const val REQUEST_LOGIN = 1001
        private const val REQUEST_START_SHIFT = 1002
        private const val REQUEST_NFC_ORDER = 1003
        private const val REQUEST_UPDATE_ORDER = 1004
        private const val REQUEST_TRANSACTION_DETAILS = 1005
    }
    
    private var isLoggedIn = false
    private var isShiftOpen = false
    
    fun initiateLogin(phone: String, password: String) {
        val intent = Intent("com.fuelin.attendant.LOGIN").apply {
            addCategory(Intent.CATEGORY_DEFAULT)
            putExtra("phone", phone)
            putExtra("password", password)
        }
        activity.startActivityForResult(intent, REQUEST_LOGIN)
    }
    
    fun startShift() {
        if (!isLoggedIn) {
            throw IllegalStateException("Must be logged in before starting shift")
        }
        
        val intent = Intent("com.fuelin.attendant.START_SHIFT").apply {
            addCategory(Intent.CATEGORY_DEFAULT)
        }
        activity.startActivityForResult(intent, REQUEST_START_SHIFT)
    }
    
    fun processNfcTag(nfcTagSerial: String) {
        if (!isLoggedIn || !isShiftOpen) {
            throw IllegalStateException("Must be logged in and have an open shift")
        }
        
        val intent = Intent("com.fuelin.attendant.NFC_ORDER").apply {
            addCategory(Intent.CATEGORY_DEFAULT)
            putExtra("nfc_tag", nfcTagSerial)
        }
        activity.startActivityForResult(intent, REQUEST_NFC_ORDER)
    }
    
    fun updateOrderQuantity(orderId: Int, quantity: String) {
        if (!isLoggedIn || !isShiftOpen) {
            throw IllegalStateException("Must be logged in and have an open shift")
        }
        
        val intent = Intent("com.fuelin.attendant.UPDATE_ORDER").apply {
            addCategory(Intent.CATEGORY_DEFAULT)
            putExtra("order_id", orderId)
            putExtra("quantity", quantity)
        }
        activity.startActivityForResult(intent, REQUEST_UPDATE_ORDER)
    }
    
    fun getTransactionDetails(orderId: Int) {
        if (!isLoggedIn) {
            throw IllegalStateException("Must be logged in to access transactions")
        }
        
        val intent = Intent("com.fuelin.attendant.TRANSACTIONS").apply {
            addCategory(Intent.CATEGORY_DEFAULT)
            putExtra("order_id", orderId)
        }
        activity.startActivityForResult(intent, REQUEST_TRANSACTION_DETAILS)
    }
    
    fun handleActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        if (resultCode != Activity.RESULT_OK || data == null) {
            // Handle cancellation or error
            return
        }
        
        when (requestCode) {
            REQUEST_LOGIN -> {
                // Process login result
                isLoggedIn = true
                // Check if shift needs to be started
            }
            
            REQUEST_START_SHIFT -> {
                // Process shift start result
                isShiftOpen = true
            }
            
            REQUEST_NFC_ORDER -> {
                // Process NFC order result
                val jsonResponse = data.getStringExtra("response_data")
                // Parse and handle the JSON response
            }
            
            REQUEST_UPDATE_ORDER -> {
                // Process order update result
            }
            
            REQUEST_TRANSACTION_DETAILS -> {
                // Process transaction details result
            }
        }
    }
}
```
