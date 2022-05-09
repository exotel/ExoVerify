# ExoVerify

This repository contains code of a sample android application where the Exotel nOTP SDK is implemented.

Note: You need to add credentials to the variables declared in the MainActivity.java to get the code working
Exoverify SDK is a native java sdk for andriod only, we don't support kotlin, flutter or react native sdks.

### Overview
Verify user’s mobile number quickly and seamlessly using nOTP verification library (Android only).

NOTP SDK automatically intercepts a phone call triggered by the nOTP system for the mobile number verification, allowing you to verify your users with no user interaction. Using the SDK, you can have nOTP verification into your existing android apps with just a few lines of code. 

A typical missed call verification would look like this:

![nOTP Flow](https://github.com/exotel/ExoVerify/blob/master/app/readme-pictures/notp_flow.png)

### nOTP Verification Process
The below flow diagram explains a typical verification process:

![nOTP Verification Process](https://github.com/exotel/ExoVerify/blob/master/app/readme-pictures/notp_verification_process.png)

### nOTP Sequence Diagram 
The below squence diagram explains the sequence of opeartions between client server and nOTP SDK and Backend :

![nOTP Sequence Diagram](https://github.com/exotel/ExoVerify/blob/master/app/readme-pictures/SequenceDiagramSDK.png)

### Pre-requisites
* User needs to have a KYC verified account with Exotel.
* The SDK can be used with Android 4.1+ versions.
* The compileSdkVersion of your app needs to be 28+.
* For verification, the phone number should be passed in E.164 format, prefixed with the plus sign (+).
* Following details provided by nOTP team are required to complete the integration of SDK & to test the verifications:
  1. Application_ID
  2. Application_Secret

Please visit [Exoverify Dashboard](https://verify.exotel.com) to generate your Application_ID and Application_Secret

## Android SDK Integration Steps
1. Include the NOTP SDK library in your Android Project:
If you are using Gradle, you need to add the libray in the app level Gradle file, as shown below:    
    
        dependencies { implementation 'org.bitbucket.Exotel:verificatrixandroid:1.6.0' }
    
2. A few other dependencies that you need to add are:
       
       implementation 'com.squareup.okhttp3:okhttp:3.6.0'
       implementation 'com.google.code.gson:gson:2.8.0'
       implementation 'com.googlecode.libphonenumber:libphonenumber:8.8.3'
       implementation 'dnsjava:dnsjava:2.1.6’
       
3. Add the following jitpack dependency in the project level Gradle file:

        allprojects {
          repositories {
            ...
            maven {
              url 'https://jitpack.io'
              credentials  { username 'jp_etcct006nc8pkd0ntra0n5uk9k' }
            }
          }
        }

4. The below permissions are required to be added in your AndroidManifest file:
                 
       <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
       <uses-permission android:name="android.permission.INTERNET" />
       <uses-permission android:name="android.permission.CALL_PHONE" />
       <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
       <uses-permission android:name="android.permission.READ_CALL_LOG"/>

5. Request for the user for the above mentioned permissions, as nOTP SDK requires these permissions in-order to verify the mobile numbers. 


### Note
nOTP verification to completely function on devices running  android version Pie (9) & above requires the following permission :

        <uses-permission android:name="android.permission.READ_CALL_LOG"/>

Above mentioned is one of the permissions that requires use-case (how this permission would be used) declaration as per updated [Google Play Policies](https://support.google.com/googleplay/android-developer/answer/9047303?hl=en).

Upon submission of your App to Google play, make sure to check the exact use-case in declaration form as shown in the below screenshot :

![nOTP Google Play Declaration](https://github.com/exotel/ExoVerify/blob/master/app/readme-pictures/notp_read_call_log_reason.png)

### JAVA Class Integration
6. Declare the ExotelVerification object in the activity where the verification is to take place:

        ExotelVerification eVerification;

7. Create a Config object using ConfigBuilder. Initialize Verification by passing the config object to the constructor:

        Config config = new ConfigBuilder(<YOUR NOTP APPID>, <YOUR NOTP APP SECRET>, <YOUR ACCOUNTSID>, getApplicationContext()).Build();
        eVerification = new ExotelVerification(config);

NOTE: You should get your AccountSid, ApplicationID and Secret key by contacting nOTP team.

8. Create a class that implements ‘VerificationListener’. This is where you can define the actions that take place when the verification of the number succeeds or fails :

        class verifyListener implements VerificationListener {
            public void onVerificationStarted(VerificationStart verificationStart) {
                Log.i(LOGGING_TAG, "Verification Started!");  
            }   
            public void onVerificationSuccess(VerificationSuccess verificationSuccess) {
                Log.i(LOGGING_TAG, "Verification Successful!");
            }
            public void onVerificationFailed(VerificationFailed verificationFailed) {
                Log.i(LOGGING_TAG, "Verification Failed: "+verificationFailed.getRequestID()+ " "+verificationFailed.getErrorCode()+" "+verificationFailed.getErrorMessage()+" "+verificationFailed.getMiscData() ); 
            }
        }

9. Start Verification:
Start the verification process by using the startVerification method. This method takes the following parameters:
* listener - new verifyListerner created in sStep 8
* phone number - number should be in E164 
* HTTP timeout - timeout value in seconds.


        eVerification.startVerification(new verifyListener(), phoneNumber, timeOutValueInSeconds);

10. Getting time in seconds after which the verification will time out. (Optional)
* Import the Timer class and TimerListener interface.
            
        import com.exotel.verification.Timer;
        import com.exotel.verification.TimerListener;

* Create an instance of the Timer class and implement the TimerListener.

        Timer customTimer = new Timer();
        customTimer.setTimerListener(new TimerListener() {
 	    @Override
        public void getTimerTick(long time) {
            secondsTv.setText("Please expect the verification call in " + String.valueOf(time/1000) + " seconds.");
        }
        });

### Troubleshooting
* Make sure you use the latest version of the SDK for best performance and security
* Try logging the results from the verification listener callbacks and try debugging the issue
* Make sure all checked Exceptions are caught and permissions are given
* Following exceptions are thrown from nOTP Verification SDK:
    1. ClientBuilderException - Occurs when building a verification client and the config is wrong. Please build the config with all fields set.
    2. InvalidConfigException - Config passed with initialization is invalid. Check if all fields are set.
    3. PermissionNotGrantedException - Necessary permissions are not granted for nOTP to work.
    4. VerificationAlreadyInProgressException - There is an ongoing verification request which did not reach a terminal state yet.
* If Client App is using progaurd , Add this to the [proguard-rules.pro](proguard-rules.pro) file in the application: 

        -keep class org.xbill.DNS.ResolverConfig
        -keep class org.xbill.DNS.Lookup
        -keep class com.exotel.** { *; }

* Fail code and messages when verification fails:

    * 801 - The config for ExotelVerification is null. Check config or initialize properly
    * 802 - The ExotelVerification request has been timed out. Call was not received on this phone.
    * 803 - The connection to the server timed out
    * 804 - Not able to establish a connection to the server
    * 805 - Auth failure error
    * 806 - Server error occurred. Please report this
    * 807 - Network error
    * 808 - Error parsing the response / Invalid response from the server. Please report this
    * 809 - API throttle limit for this application has been reached. Please wait for it to reset
    * 810 - Throttle limit for this phone number has been reached. Please wait for it to reset
    * 811 - Unknown Error occurred. Please report this.
    * 812 - Phone might have been busy on another call during verification. Or it's a timeout
    * 813 - Not able to get Cell Signal/ No country Code returned
    * 814 - Invalid Number/ not in E164 format

Contact nOTP support with supporting logs if issue persists or requires further support.

### CallBacks & Webhooks : 
Both these webhooks are optional , There are two callbacks that we support:

* The Verification confirmation webhook: <p> Before the exoverify service makes a call, it first hits the endpoint that the tenant has
configured to see if we need to continue to make the call or not. If the tenant returns a
200, we proceed. 
If not, we fail the verification with the reason `Denied`. This webhook
is optional and is only hit if it is configured but the client. 
  * The sample payload for the webhook is:

        {
        "verification_id": "5f564a2edd154e95b752cc260ead3b3f",
        "application_id":"2d3a8f96d9e7436a9a5f93cae85dd3",
        "country_code":"IN",
        "mobile_number":"+919910166288",
        }        

   * Default Method supported : <b>GET</b>
</p>     

* Timeouts: <p> Default Timeout = 15 seconds (Your servers should ideally consume verification status within milliseconds, but Exotel will wait a maximum of 15 seconds for your endpoint to consume verification status and return `200 HTTP code`).
In such cases,after 15 seconds we fail the particular verification request with status ` Denied` ,
If your server needs to perform any complex logic or make internal network calls, it is possible that the verification status webhook request might time out. For that reason, you might want to have your webhook endpoint immediately acknowledge receipt by returning a `200 HTTP status code `, and then perform the rest of its operations, async.
</p>

* The Verification status webhook: <p> This webhook will ping the status of each verification request to your configured url endpoint. Already the NOTP Android SDK would be updating the application on status of the verification, 
this is an additional mechanism for the tenant’s backend system to capture the status of verification request easily. 
  * Sample for when the verification was <b>successful</b>:

        {
        "verification_id": "5f564a2edd154e95b752cc260ead3b3f",
        "application_id":"2d3a8f96d9e7436a9a5f93cae85dd3",
        "country_code":"IN",
        "mobile_number":"+919910166288",
        "status":"success",
        "reason":null,
        "timestamp": "2018-08-22T15:19:23Z",
        }


  * Sample for when the verification <b>failed</b>:

        {
        "verification_id":"5f564a2edd154e95b752cc260ead3b3f",
        "application_id":"2d3a8f96d9e7436a9a5f93cae85dd3",
        "country_code":"IN",
        "mobile_number":"+919910166288",
        "status":"fail",
        "reason":"timeout",
        "timestamp": "2018-08-22T15:19:23Z",
        }
  * Default Method supported : <b>POST</b>

</p>

* Responding to this webhook: <p> The HTTP Response for acknowledging the verification status callback webhook
request should be 200 OK. All other non-200 HTTP codes or if your server is unable to
respond will indicate that you have not received the verification. In such cases we don’t
retry.</p>