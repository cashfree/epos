# Cashfree SDK Integration Steps

## Step 1: Download Library

The Cashfree ePOS SDK is bundled as an AAR file according to the latest Android standards. Please download and include this library as a dependency in your app. . 

## Step 2: Add Dependency

  - Create <b>libs</b> folder inside your module. <br> ex: <b>app/libs</b> should be location for <b>app</b> module
  - Copy your .aar file to this libs folder
  - Open <b>build.gradle</b> file inside the module(the one under ‘app’) and add dependency :


```xml
  dependencies {
    implementation files(‘libs/epos-sdk-1.0-release.aar)
  }
```
<br/>

If you have to use this library in multiple modules use the method mentioned [here.](https://developer.android.com/studio/projects/android-library#AddDependency)

## Step 3: Add permissions and Dependencies

The CashfreeSDK requires that you add the permissions shown below in your `Android Manifest` file.
We support integration from API level 16. Do ensure that the minSdkVersion in the build.gradle of your app is equal to (or greater than) that.

```xml
<manifest ...>
    <uses-permission android:name="android.permission.INTERNET" />
<application ...>
```

Add  these dependencies to your build.gradle file. 

```xml
dependencies {
    ...  
    //Dependencies used by all payment modes  
    implementation 'androidx.appcompat:appcompat:1.1.0'  
    implementation 'com.android.volley:volley:1.1.1'  
}
```

<br>

## Step 4: Generate Token (From Backend)
You will need to generate a token from your backend and pass it to app while initiating payments. For generating token you need to use our token generation API. Please take care that this API is called only from your <b><u>backend</u></b> as it uses **secretKey**. Thus this API should **never be called from App**.

<br/>

### Request Description

<copybox>

  For production/live usage set the action attribute of the form to:
   `https://api.cashfree.com/eposapi/v1/cftoken/order`

  For testing set the action attribute to:
   `https://test.cashfree.com/eposapi/v1/cftoken/order`

</copybox>

You need to send orderId, orderCurrency and orderAmount as a JSON object to the API endpoint and in response a token will received. Please see  the description of request below.

```bash
curl -XPOST -H 'Content-Type: application/json' 
-H 'x-client-id: <YOUR_APP_ID>' 
-H 'x-client-secret: <YOUR_SECRET_KEY>' 
-d '{
  "orderId": "<ORDER_ID>",
  "orderAmount":<ORDER_AMOUNT>,
  "orderCurrency": "INR"
}' 'https://test.cashfree.com/eposapi/v1/cftoken/order'
```
<br/>

### Request Example

Replace **YOUR_APP_ID** and **YOUR_SECRET_KEY** with actual values.
```bash
curl -XPOST -H 'Content-Type: application/json' -H 'x-client-id: YOUR_APP_ID' -H 'x-client-secret: YOUR_SECRET_KEY' -d '{
  "orderId": "Order0001",
  "orderAmount":1,
  "orderCurrency":"INR"
}' 'https://test.cashfree.com/eposapi/v1/cftoken/order'
```
<br/>

### Response Example

```bash
{
"status": "OK",
"message": "Token generated",
"cftoken": "v79JCN4MzUIJiOicGbhJCLiQ1VKJiOiAXe0Jye.s79BTM0AjNwUDN1EjOiAHelJCLiIlTJJiOik3YuVmcyV3QyVGZy9mIsEjOiQnb19WbBJXZkJ3biwiIxADMwIXZkJ3TiojIklkclRmcvJye.K3NKICVS5DcEzXm2VQUO_ZagtWMIKKXzYOqPZ4x0r2P_N3-PRu2mowm-8UXoyqAgsG"
}
```

The "cftoken" is the token that is used authenticate your payment request that will be covered in the next step.
<br/>

## Step 5: Initiate Collect Request

- App passes the order info and the token to the SDK.
- Agent is shown the appropriate screen
- Once the payment is complete SDK verifies the payment
- App receives the response from SDK and handles it appropriately


### NOTE

- The order details passed during the token generation and the payment initiation should match. Otherwise you'll get a<b>"invalid order details"</b> error.
- Wrong appId and token will result in <b>"Unable to authenticate merchant"</b> error. The token generated for payment is valid for 5 mins within which the payment has to be initiated. Otherwise you'll get a <b>"Invalid token"</b> error.
  
# Payment modes


## QR

```java

void collectQRPayment(Activity context, Map<String, String> params,
                          String token, String stage);

```

Creates an order on the cashfree server and shows a screen which has the UPI QR code. This can be scanned from any (supported) UPI app on the customer's phone. 

<b>Parameters:</b>

-  <code>context</code>: Context object of the calling activity is required for this method. In most of the cases this will mean sending the instance of the calling activity (this).

-  <code>params</code>:  A map of all the relevant parameters. Check the [request parameters](#request-params) section for details.

-  <code>token</code>: The token generated from **Step 4**.

-  <code>stage</code>: Value should be either "**TEST**" or "**PROD**" for testing server or production server respectively.

  
  

### Remote Link

```java

void collectRemotePayment(Activity context, Map<String, String> params,
                                                         String token, String stage)

```

Creates an order on the cashfree server and sends a payment link to the customer via SMS and email. 

  
  

<b>Parameters:</b>

  

-  <code>context</code>: Context object of the calling activity is required for this method. In most of the cases this will mean sending the instance of the calling activity (this).

-  <code>params</code>: A map of all the relevant parameters. Check the [request parameters](#request-params) section for details.

-  <code>token</code>: The token generated from **Step 4**.

-  <code>stage</code>: Value should be either "**TEST**" or "**PROD**" for testing server or production server respectively.

  
    

## QR and RemoteLink

```java

void collectPayment(Activity context, Map<String, String> params,
                        String token, String stage);

```

  Opens a screen from the SDK which shows both the payment collection modes. Once the agent selects a payment mode as per the customer's preference, the SDK creates an order on the cashfree server and shows the appropriate screen for the selected mode.

  

<b>Parameters:</b>

-  <code>context</code>: Context object of the calling activity is required for this method. In most of the cases this will mean sending the instance of the calling activity (this).

-  <code>params</code>:  A map of all the relevant parameters. Check the [request parameters](#request-params) section for details.

-  <code>token</code>: The token generated from **Step 4**.

-  <code>stage</code>: Value should be either "**TEST**" or "**PROD**" for testing server or production server respectively.
 


## Receive Response

  

Once the payment is done you will receive the response on the onActivityResult() function inside the invoking activity. In the intent extras you will receive a set of response parameters which you can use to determine if the transaction was successful. Request code will always be equal to CFPaymentService.REQ_CODE.

  

```java

@Override

protected  void  onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        //Same request code for all payment APIs.
        Log.d(TAG, "API Response : ");
        //Prints all extras. Replace with app logic.
        if (data != null) {
            Bundle  bundle = data.getExtras();
            if (bundle != null)
                for (String  key  :  bundle.keySet()) {
                    if (bundle.getString(key) != null) {
                        Log.d(TAG, key + " : " + bundle.getString(key));
                    }
                }
        }
}
```

<br/>

## Request Params



| Parameter                                 | Required | Description                                      |
|-------------------------------------|-----------|----------------------------------------------------|
| <code>appId</code>            | Yes      | Your app id      |
| <code>orderId</code> | Yes       | Order/Invoice Id  |
| <code>orderAmount</code> | Yes       | Bill amount of the order      |
| <code>orderCurrency</code> | Yes       | Currency code. Default is INR      |
| <code>orderNote</code>            | No       | A help text to make customers know more about the order                                |
| <code>agentAlias</code> | Yes    | Name of the agent (Valid)     |
| <code>customerPhone</code> | Yes    | Phone number of customer     |
| <code>customerEmail</code> | Yes    | Email id of the customer     |
| <code>notifyUrl</code> | No    | Notification URL for server-server communication. Useful when user’s connection drops after completing payment.     |


## Response Params

These parameters are sent as extras to the onActivityResult(). They contain the details of the transaction.

| Parameter                                  | Description                                      |
|------------------------------------------------|----------------------------------------------------|
| <code>orderId</code>  | Order id for which transaction has been processed. Ex: GZ-212  |
| <code>orderAmount</code> | Amount of the order. Ex: 256.00      |
| <code>paymentMode</code> | Payment mode of the transaction.      |
| <code>referenceId</code>      | Cashfree generated unique transaction Id. Ex: 140388038803                                |
| <code>txStatus</code>   | Payment status for that order. Values can be : SUCCESS, FLAGGED, PENDING, FAILED, CANCELLED.     |
| <code>txMsg</code>   | Message related to the transaction. Will have the reason, if payment failed     |
| <code>txTime</code>  | Time of the transaction    |
| <code>signature</code>  | Response signature, more [here.](https://docs.cashfree.com/pg/cf-checkout/#response-verification)    |


## Sample Code

```java
    //Change the values inside <> to real values
    Map<String, String> params = new HashMap<>();
    params.put(PARAM_APP_ID, <APP_ID>);
    params.put(PARAM_ORDER_ID, "order1");
    params.put(PARAM_ORDER_AMOUNT, "1");
    params.put(PARAM_ORDER_NOTE, "Cashfree Test");
    params.put(PARAM_CUSTOMER_NAME, "Cashfree");
    params.put(PARAM_AGENT_ALIAS, "AgentAlias");
    params.put(PARAM_CUSTOMER_PHONE, <VALID_PHONE_NO>);
    params.put(PARAM_CUSTOMER_EMAIL,<VALID_EMAIL_ID>);
    params.put(PARAM_NOTIFY_URL,"https://www.yourendpoint.com/");
    params.put(PARAM_ORDER_CURRENCY, orderTokenApi.getOrderCurrency());
    
    String stage = <TEST or PROD>;
    String token = <cftoken generated for this order>
    
    CFCollectAPI cfPaymentService = CFPOSService.getCFPOSServiceInstance();
    //Replace with the SDK function according to your requirement
    cfPaymentService.collectPayment(CFePosActivity.this, params, token, stage);
```

