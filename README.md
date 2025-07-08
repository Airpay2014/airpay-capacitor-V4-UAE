# Airpay Ionic Capacitor Plugin

This is the official Ionic Capacitor plugin for integrating the Airpay payment gateway into Ionic applications.

## Supported Platforms

* **Android** (version compatibility details below)  
  - Up to Gradle version 8.7

* **iOS** 
  - Xcode 16.2_swiftUI  

## Requirements

* **Node.js Version:** 22.11.0
* **Ionic Version:** 8.5.0
* **Ionic CLI:** 7.2.0
* **Capacitor CLI:** 7.1.0
* **@capacitor/android:** 7.1.0
* **@capacitor/core:** 7.1.0
* **npm:** 8.19.4

## Installation

> **Note:** For Windows users, please run the following commands in Git Bash instead of Command Prompt. You can download Git for Windows [here](https://git-scm.com/downloads).

### Install Ionic CLI and Capacitor CLI

```sh
npm install -g @ionic/cli
npm install -g @capacitor/cli
```

### Create a New Ionic Project

```sh
ionic start ionic_sample_app blank --type=angular
cd ionic_sample_app/
```

### Enable Capacitor and Add Android Platform

```sh
ionic integrations enable capacitor
npm install @capacitor/android
npx cap add android
npx cap sync android
```

### Enable Capacitor and Add iOS Platform

```sh
npm install @capacitor/ios
npx cap add ios
npx cap sync ios
```

### Install the Airpay Plugin - (Ionic Capacitor v8)

```sh
npm install https://github.com/Airpay2014/airpay-capacitor-V4-UAE.git
ionic build
ionic cap sync android
ionic cap sync ios
```

## Android Integration

The following Android files are crucial for integrating Airpay with Capacitor:

### AndroidManifest.xml

Add the following inside the `<application>` tag:

```xml
<uses-library
    android:name="org.apache.http.legacy"
    android:required="false" />
```

Ensure internet permission is granted:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### MainActivity.java

Modify `MainActivity.java` to include Airpay imports:

```java



import com.airpay.ae.AirpayConfig;
import com.airpay.ae.constants.ConfigConstants;
import com.airpay.ae.model.data.TransactionDto;
import com.airpay.ae.utils.Utility;
import com.airpay.ae.view.ActionResultListener;
import com.airpay.plugins.mycustomplugin.AirpayPlugin;
import com.getcapacitor.BridgeActivity;
import com.getcapacitor.JSObject;

import android.annotation.SuppressLint;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.os.Handler;
import android.text.TextUtils;
import android.util.Log;
import android.view.WindowManager;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.Toast;

import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatDelegate;
import androidx.localbroadcastmanager.content.LocalBroadcastManager;

import org.json.JSONObject;

import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.zip.CRC32;

public class MainActivity extends BridgeActivity implements ActionResultListener {
  public ActivityResultLauncher<Intent> airpayLauncher;

  private boolean doubleBackToExitPressedOnce = false;

  private static final String TAG = "MainActivity";


  private AlertDialog alertDialog;
  private BroadcastReceiver receiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
      if (AirpayPlugin.LOCAL_AIRPAY_BORADCAST_EVENT.equals(intent.getAction())) {
        if (intent != null) {

          String requestData = intent.getStringExtra("request_data");
          Log.d("requestData", requestData);
          try {
            // Convert jsonString to JSONObject
            JSONObject outerJson = new JSONObject(requestData);
            // Get the "value" field which contains the actual JSON string
            String innerJsonString = outerJson.getString("value");
            // Replace escaped quotes \" with actual quotes "
            innerJsonString = innerJsonString.replace("\\\"", "\"");
            // Convert to JSONObject
            JSONObject innerJson = new JSONObject(innerJsonString);

            // Extract values
            String firstName = innerJson.getString("firstName");
            String lastName = innerJson.getString("lastName");
            String email = innerJson.isNull("email") ? "" : innerJson.getString("email");
            String address = innerJson.isNull("fullAddress") ? "" : innerJson.getString("fullAddress");
            String city = innerJson.isNull("city") ? "" : innerJson.getString("city");
            String emirate = innerJson.isNull("emirate") ? "" : innerJson.getString("emirate");
            String phone = innerJson.isNull("phone") ? "" : innerJson.getString("phone");
            String country = innerJson.isNull("country") ? "" : innerJson.getString("country");
            String pincode = innerJson.isNull("pincode") ? "" : innerJson.getString("pincode");
            String orderId = innerJson.getString("orderId");
            String amount = innerJson.getString("amount");



            DateFormat df1 = new SimpleDateFormat("yyyy-MM-dd");
            String sCurDate1 = df1.format(new Date());

            String sAllData1 = email + firstName
              + lastName + address
              + city + emirate
              + country + amount
              + orderId + sCurDate1;


            // Merchant details

            String sMid = "";
            String sSecret = "";
            String sUserName = "";
            String sPassword = "";
            String sClientId = "";
            String sClient_secret = "";
            String merdome = "";
            String successUrl = "";
            String failureUrl = "";

            // private key
            String sTemp = sSecret + "@" + sUserName + ":|:" + sPassword;
            String sPrivateKey = Utility.sha256(sTemp);

            // key for checksum
            String sTemp3 = sUserName + "~:~" + sPassword;
            String sKey1 = Utility.sha256(sTemp3);

            // checksum
            sAllData1 = sKey1 + "@" + sAllData1;
            String sChecksum1 = Utility.sha256(sAllData1);

            new AirpayConfig.Builder(airpayLauncher, MainActivity.this)
              .setActionType("IndexPay")
              .setEnvironment(ConfigConstants.STAGING)
              .setType(ConfigConstants.AIRPAY_KIT)
              .setPrivateKey(sPrivateKey)
              .setSecretKey(sSecret)
              .setMerchantId(sMid)
              .setOrderId(orderId)
              .setCurrency("784") // 784
              .setIsoCurrency("AED") //
              .setEmailId(email)
              .setMobileNo(phone)
              .setBuyerFirstName(firstName)
              .setBuyerLastName(lastName)
              .setBuyerAddress(address)
              .setBuyerCity(city)
              .setBuyerState(emirate)
              .setBuyerCountry(country)
              .setAmount(amount)
              .setWallet("0")
              .setChmod("")
              .setChecksum(sChecksum1)
              .setTxnsubtype("")
              .setMerDom(merdome)
              .setSuccessUrl(successUrl)
              .setFailedUrl(failureUrl)
              .setLanguage("EN")
              .setClient_id(sClientId)
              .setClient_secret(sClient_secret)
              .setGrant_type("client_credentials")
              .setAesDesKey(sTemp3)
              .build()
              .initiatePayment();

          } catch (Exception e) {
            e.printStackTrace();
          }
        }

      }
    }
  };

  @Override
  public void onCreate(Bundle savedInstanceState) {

    registerPlugin(AirpayPlugin.class);

    super.onCreate(savedInstanceState);
    // pluginCall = new AirpayPlugin();
    AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO);

    airpayLauncher = registerForActivityResult(
      new ActivityResultContracts.StartActivityForResult(),
      result -> {
        if (result.getResultCode() == RESULT_OK) {
          Intent data = result.getData();
          if (data != null) {
            TransactionDto transaction = (TransactionDto) data.getParcelableExtra("transaction");
            if (transaction != null) {
              Log.d("MyCustomPlugin", "Transaction status: " + transaction.getData().getTransactionStatus());
              Toast.makeText(this, "Transaction Status: " + transaction.getData().getTransactionStatus(), Toast.LENGTH_SHORT).show();

              JSObject response = new JSObject();
              response.put("STATUS", transaction.getStatus());
              response.put("STATUSMSG", transaction.getMessage());
              response.put("TXN_MODE", transaction.getData().getTxnMode());
              response.put("TRANSACTIONID", transaction.getData().getApTransactionid());
              response.put("TRANSACTIONAMT", transaction.getData().getAmount());
              response.put("TRANSACTIONSTATUS", transaction.getData().getTransactionStatus());
              response.put("MERCHANTTRANSACTIONID", transaction.getData().getMerchantId());
             // response.put("MERCHANTPOSTTYPE", transaction.getMERCHANTPOSTTYPE());
              response.put("MERCHANTKEY", transaction.getData().getMerchantId());
              response.put("SECUREHASH", transaction.getData().getApSecurehash());
              response.put("CUSTOMVAR", transaction.getData().getCustomVar());
              response.put("TXN_DATE_TIME", transaction.getData().getTransactionTime());
              response.put("TXN_CURRENCY_CODE", transaction.getData().getCurrencyCode());
              response.put("TRANSACTIONVARIANT", transaction.getData().getTransactionType());
              response.put("CHMOD", transaction.getData().getChmod());
              response.put("bankResponseMsg", transaction.getData().getBankResponseMsg());
              response.put("transactionPaymentStatus", transaction.getData().getTransactionPaymentStatus());
            //  response.put("BANKNAME", transaction.getBANKNAME());
             // response.put("CARDISSUER", transaction.getCARDISSUER());
              response.put("FULLNAME", transaction.getData().getCustomerName());
              response.put("EMAIL", transaction.getData().getCustomerEmail());
              response.put("CONTACTNO", transaction.getData().getCustomerPhone());
              response.put("ISRISK", transaction.getData().getRisk());
            //  response.put("MERCHANT_NAME", transaction.getMERCHANT_NAME());
             // response.put("SETTLEMENT_DATE", transaction.getSETTLEMENT_DATE());
              response.put("SURCHARGE", transaction.getData().getSurchargeAmount());
             // response.put("BILLEDAMOUNT", transaction.getBILLEDAMOUNT());
           //   response.put("CUSTOMERVPA", transaction.getCUSTOMERVPA());

              String orderId = transaction.getData().getOrderid();
              String apTransactionID = transaction.getData().getApTransactionid();
              String amount = transaction.getData().getAmount();
              int transtatus = transaction.getData().getTransactionStatus();
              String transactionPaymentStatus = transaction.getData().getTransactionPaymentStatus();



              String merchantid = ""; //Please enter Merchant Id
              String username = "";	  //Please enter Username
			  
              String sParam = orderId + ":" + apTransactionID + ":" + amount + ":" + transtatus + ":" + transactionPaymentStatus + ":" + merchantid + ":" + username;
              CRC32 crc = new CRC32();
              crc.update(sParam.getBytes());
              String sCRC = "" + crc.getValue();
              Log.e("Verified Hash ==", "sParam= " + sParam);
              Log.e("Verified Hash ==", "Calculate Hash= " + sCRC);
              Log.e("Verified Hash ==", "RESP Secure Hash= " + transaction.getData().getApSecurehash());

              if (sCRC.equalsIgnoreCase(transaction.getData().getApSecurehash())) {
                Log.e("Verified Hash ==", "SECURE HASH MATCHED");
              } else {
                Log.e("Verified Hash ==", "SECURE HASH MIS-MATCHED");
              }

              AirpayPlugin.resolvePaymentResult(response.toString());

            } else {
              Log.e("MyCustomPlugin", "Transaction object is null");
              AirpayPlugin.rejectPayment("Transaction object is null");
            }
          } else {
            Log.e("MyCustomPlugin", "Intent data is null");
            AirpayPlugin.rejectPayment("Payment failed: No data received");
          }
        } else {
          Log.e("MyCustomPlugin", "Payment failed, Result Code: " + result.getResultCode());
          AirpayPlugin.rejectPayment("Payment failed, Result Code: " + result.getResultCode());
        }
      }
    );


  }

  @SuppressLint("MissingSuperCall")
  @Override
  public void onBackPressed() {
    if (doubleBackToExitPressedOnce) {
      finishAffinity(); // Closes the app
      return;
    }

    this.doubleBackToExitPressedOnce = true;
    Toast.makeText(this, "Press back again to exit", Toast.LENGTH_SHORT).show();

    new Handler().postDelayed(() -> doubleBackToExitPressedOnce = false, 2000);

  }

  public String appendDecimal(String input) {
    if (input == null || input.isEmpty()) {
      return "0.00"; // Default value for empty input
    }

    // Check if input already has a decimal
    if (!input.contains(".")) {
      return input + ".00"; // Append .00 if no decimal exists
    }

    return input; // Return as-is if it already has a decimal
  }

  @Override
  public void onResume() {
    super.onResume();
    LocalBroadcastManager.getInstance(this)
      .registerReceiver(receiver, new IntentFilter(AirpayPlugin.LOCAL_AIRPAY_BORADCAST_EVENT));
  }

  @Override
  public void onPause() {
    super.onPause();
    LocalBroadcastManager.getInstance(this).unregisterReceiver(receiver);
  }

  @Override
  public void onResult(Object o) {
    if (o instanceof TransactionDto) {
      TransactionDto transaction = (TransactionDto) o;

      Toast.makeText(MainActivity.this, transaction.getData().getTransactionStatus() + "\n" + transaction.getMessage(), Toast.LENGTH_LONG).show();
      if (transaction.getStatus() != null) {
        Log.e("STATUS -> ", "=" + transaction.getStatus());
      }
      if (transaction.getData().getMerchant_key() != null) {
        Log.e("MERCHANT KEY -> ", "=" + transaction.getData().getMerchant_key());
      }
      if (transaction.getData().getAmount() != null) {
        Log.e("TRANSACTION AMT -> ", "=" + transaction.getData().getAmount());
      }
      if (transaction.getData().getTxnMode() != null) {
        Log.e("TXN MODE -> ", "=" + transaction.getData().getTxnMode());
      }
      if (transaction.getData().getApTransactionid() != null) {
        Log.e("MERCHANT_TXN_ID -> ", "=" + transaction.getData().getApTransactionid()); // order id
      }
      if (transaction.getData().getApSecurehash() != null) {
        Log.e("SECURE HASH -> ", "=" + transaction.getData().getApSecurehash());
      }
      if (transaction.getData().getCustomVar() != null) {
        Log.e("CUSTOMVAR -> ", "=" + transaction.getData().getCustomVar());
      }
      if (transaction.getData().getApTransactionid() != null) {
        Log.e("Order ID -> ", "=" + transaction.getData().getApTransactionid());
      }
      if (transaction.getData().getTransactionStatus() != 0) {
        Log.e("TXN STATUS -> ", "=" + transaction.getData().getTransactionStatus());
      }
      if (transaction.getData().getTransactionTime() != null) {
        Log.e("TXN_DATETIME -> ", "=" + transaction.getData().getTransactionTime());
      }
      if (transaction.getData().getCurrencyCode() != null) {
        Log.e("TXN_CURRENCY_CODE -> ", "=" + transaction.getData().getCurrencyCode());
      }

      if (transaction.getData().getChmod() != null) {
        Log.e("CHMOD -> ", "=" + transaction.getData().getChmod());
      }

      if (transaction.getData().getCustomerName() != null) {
        Log.e("FULLNAME -> ", "=" + transaction.getData().getCustomerName());
      }
      if (transaction.getData().getCustomerEmail() != null) {
        Log.e("EMAIL -> ", "=" + transaction.getData().getCustomerEmail());
      }
      if (transaction.getData().getCustomerPhone() != null) {
        Log.e("CONTACTNO -> ", "=" + transaction.getData().getCustomerPhone());
      }

      if (transaction.getData().getTransactionTime() != null) {
        Log.e("SETTLEMENT_DATE -> ", "=" + transaction.getData().getTransactionTime());
      }
      if (transaction.getData().getSurchargeAmount() != null) {
        Log.e("SURCHARGE -> ", "=" + transaction.getData().getSurchargeAmount());
      }
      if (transaction.getData().getRisk() != null) {
        Log.e("ISRISK -> ", "=" + transaction.getData().getRisk());
      }
      String orderId = transaction.getData().getOrderid();
      String apTransactionID = transaction.getData().getApTransactionid();
      String amount = transaction.getData().getAmount();
      int transtatus = transaction.getData().getTransactionStatus();
      String transactionPaymentStatus = transaction.getData().getTransactionPaymentStatus();



      String merchantid = ""; //Please enter Merchant Id
      String username = "";   //Please enter username
      String sParam = orderId + ":" + apTransactionID + ":" + amount + ":" + transactionPaymentStatus + ":" + transtatus + ":" + merchantid + ":" + username;
      CRC32 crc = new CRC32();
      crc.update(sParam.getBytes());
      String sCRC = "" + crc.getValue();
      Log.e("Verified Hash ==", "sParam= " + sParam);
      Log.e("Verified Hash ==", "Calculate Hash= " + sCRC);
      Log.e("Verified Hash ==", "RESP Secure Hash= " + transaction.getData().getApSecurehash());

      if (sCRC.equalsIgnoreCase(transaction.getData().getApSecurehash())) {
        Log.e("Verified Hash ==", "SECURE HASH MATCHED");
      } else {
        Log.e("Verified Hash ==", "SECURE HASH MIS-MATCHED");
      }
    }


  }

  @Override
  public void onFailure(String s) {
    Toast.makeText(this, s, Toast.LENGTH_SHORT).show();
  }

}



```

### build.gradle (app-level)

Add the following dependencies:

```gradle

implementation("com.airpay:airpay-uae-kit:1.0.6"){
    exclude group: 'androidx.core', module: 'core'
    // or
    exclude group: 'androidx.legacy', module: 'legacy-support-v4'
  }
  implementation 'androidx.multidex:multidex:2.0.1'
  
```

### build.gradle (project-level)

Add the following repository configurations: 
Note:- For the username and password , please refer the ionic capacitor kit on sanctum link.  

```gradle
maven { url 'https://maven.google.com' }
  maven {
      url "https://gitlab.com/api/v4/projects/68861779/packages/maven"
      name = "GitLab"
      credentials {
        username = ""
        password = ""
      }
    }
```

### gradle.properties

Add the following properties:

```properties
android.enableJetifier=true
org.gradle.java.home=C:\\Program Files\\Java\\jdk-21
```

### AirpayPlugin.java

Update the following fields with your merchant configuration details:

```java
// Merchant details
String sMid = ""; // Enter Merchant ID
String sSecret = ""; // Enter Secret Key
String sUserName = ""; // Enter Username
String sPassword = ""; // Enter Password
String client_id = ""; //please enter client id
String client_secret = ""; //please enter client secret

.setSuccessUrl("") // Enter Success URL
.setFailedUrl("") // Enter Failed URL
.setMerDom("") //Enter Success URL Domain
```

### Angular Code - home.page.ts

Example of calling the `.open` function on button click:

```typescript
await MyCustomPlugin.open({ value: jsonData })
    .then((result) => {
        alert("Return value is " + result.value);
    })
    .catch((error) => {
        alert("Error is " + error);
    });
```


## iOS Integration

The following iOS files are crucial for integrating Airpay with Capacitor:

### Adding Framework
Need to change the general setting -(Go to project settings -> general -> Frameworks, libraries, and Embedded Content -> Select the library and set Embed & sign)

Please refer to the Ionic Capacitor sample kit code available on the Sanctum portal for integration, which includes the Airpay Framework. Ensure the following changes are applied to your project.

### Info.plist - Adding below code 

```
<key>LSApplicationQueriesSchemes</key>
    <array>
      <string>phonepe</string>
      <string>gpay</string>
      <string>bhim</string>
      <string>paytmmp</string>
      <string>amazonToAlipay</string>
      <string>whatsapp</string>
      <string>credpay</string>
      <string>mobikwik</string>
    </array>
```
	
### AirpayDemoViewModel.swift file changes -


Need to configured the Merchant Configuration details inside the AirpayDemoViewModel.swift file -

```
    @Published var kAirPaySecretKey: String = "" //(Enter the Secret Key value)
    @Published var kAirPayUserName: String = ""  //(Enter the Username value)
    @Published var kAirPayPassword: String = ""  //(Enter the Password value)
    @Published var successURL: String = ""       //(Enter the Success url value)
	@Published var merchantID:String = ""        //(Enter the Merchant Id value)
	@Published var client_secret:String = ""   //(Enter the Client_secret Id value)
	@Published var client_id:String = ""	   //(Enter the Client Id value)	
	
```
	
Refer the PrivateKey function -

```
 func privateKey()->String {
      let sTemp = "\(kAirPaySecretKey)\("@")\(kAirPayUserName)\(":|:")\(kAirPayPassword)"
      let hashCode1 = "\(sTemp)"
      let sPrivateKey = hashCode1.sha256Hash()
      print("sPrivateKey: \(sPrivateKey)")
      return sPrivateKey
  }
```
  
Refer the Checksum Calculation function inside the AirpayDemoViewModel.swift
 
```

 func getCheckSum(privateKey:String, currentDate:String,email:String,firstName:String,lastName:String,address:String,city:String,state:String,country:String,orderID:String,amount:String) -> String {
    
      let stringAll = "\(email)\(firstName)\(lastName)\(address )\(city)\(state)\(country)\(amount)\(orderID)\(currentDate)"
      
      let sTemp2 = "\(kAirPayUserName)\("~:~")\(kAirPayPassword)"
      let hashCode2 = "\(sTemp2)"
      let sKey = hashCode2.sha256Hash()
      
      let sAllData = "\(sKey)@\(stringAll)"
      let checksumStr = "\(sAllData.sha256Hash())"
      
     
      return checksumStr
  }
  
```	
(Validation of fields , checksum , private key, date calculation, encodeDomainToBase64 , sha256Hash all are functions logics mentioned inside the AirpayDemoViewModel.swift class)  

### Response handling will be managed by the finishPayment() method - 

#### Note :- AirPayDelegate class was extended to the AirpayDemoViewModel.swift class hence we are able to get the finishPayment method on AirpayDemoViewModel.swift class


Note:- Securehash logic was mentioned inside the finishPayment method. Calculated securehash will be matched with securehash getting from the transaction response, By matching the securehash value we can validate the transaction response getting from the server.

If the securehash is mismatched then the reponse which received from server is inproper for the requested transaction id.


### AppDelegate.swift file changes -

In this class, handleAirPayNotification method contains the logic to send the request parameters to the framework and also handling the response to the ionic app side using notification centre.

For detailed instructions and examples, please refer to the “SampleIonicCapacitor App” and the “Documents” folder included in the kit package.(Kindly refer Sanctum Portal Link)
 

> **Note:** For merchant configuration details, please contact the Airpay Support Team.

## Help

For troubleshooting, use:

```sh
npx cap doctor
```

## Support

For assistance, please contact:

* **Technical/Integration Support Team**
* **Customer Support Team**

## Version History

* **1.0** - Initial Release

## License

This project is licensed under the [Airpay Payment Service] License.

