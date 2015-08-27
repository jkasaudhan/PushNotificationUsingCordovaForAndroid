# PushNotificationUsingCordovaForAndroid
Implemented and tested sample project for push notification using cordova for android
Date : 20th March 2014

How to implement push notification using cordova ?

I have implemented push notification for android with 
cordova : version 3.3.0-rc.1,
Mobile : Nexus 4

Implementation on client app :

step 1) create new project on cloud.google.com/console/project and enable Google Cloud Messaging Service.
	Note down the project ID eg. 941079645636 in my case -- used in client side.
	Note down api-key =" AIzaSyDdgcgIC67lkdkKKld37tnw6BlsS_neqc"  which is used by our local php server.

step 2) create new cordova project, build and run it.

	cordova create SimplePushNotificationCordova
	cd SimplePushNotificationCordova
	cordova platform add android
	cordova build android
	cordova run android

step 3)open following github link
	https://github.com/phonegap-build/PushPlugin

	navigate to cordova project and use
	cordova plugin add https://github.com/phonegap-build/PushPlugin.git

step 4) open SimplePushNotificationCordova/www/index.html
	edit 
	 <p class="event received" onClick="init();">Register Device </p>
	and add following functions
	
	function init(){
			alert("Calling function to register mobile device on GCM");
			window.plugins.pushNotification.register(
			successHandler,
			errorHandler,
			{"senderID":"941079645636",
			 "ecb":"onNotificationGCM"
			});	
		}
		function successHandler(result){
		alert("Result" + result);		
		}
		function errorHandler(error){
		alert("error"+ result);
		}
		function onNotificationGCM(e){
		alert("Message : " + e.event );
		switch(e.event){
			case 'registered':
				alert("ID: " + e.regid);
				sendRequest(e.regid);
				alert("Successfully Registered");
				break;
			case 'message':	
				alert("Message: "+ e.payload.message);
				var sound = new Media("assets/www/"+e.soundname);
				sound.play();	
				break;
			default:
				alert("unknown event");
		}

		}
		//use to send registration ID returned by GCM to our local php server
		//regID should be save in our local database so that we can use regID and api-key from google to sent request to GCM
		//from php server in mycase.
		function sendRequest(regID){
		alert("sending request ..");
		$.post("http://192.168.1.70/phpgcmserver/register.php",
			{
			regID:regID,
			name:"jitendra",
			email:"jiten.ktm@gmail.com"
			});
		}
step 5) Inside project directory
	SimplePushNotificationCordova> cordova run android 
	
	(NOTE: 
		Push Notification cannot be tested in emulator.
		Cordova does not provide built-in api for push notification that's why, we are using plugin which have java code beneath it.
		Do not forget to check if internet is working in mobile device,internet should be working.
		Make sure localhost is wokring, http://192.168.1.70/phpgcmserver/register.php in my case.

	)	
step 6) Develop server app in php to store regID and send request to GCM.
	public function send_notification($registatoin_ids, $message) {
        // include config
        define("DB_HOST","localhost");
	define("DB_USER","root");
	define("DB_PASSWORD","12345");
	define("DB_DATABASE","gcm");
	
	define("GOOGLE_API_KEY","AIzaSyDdgcgIC67lkdkKKld37tnw6BlsS_neqc");
	define("SENDER_ID","941079645636");
 
        // Set POST variables
        $url = 'https://android.googleapis.com/gcm/send';
 
        $fields = array(
            'registration_ids' => $registatoin_ids,
            'data' => $message,
        );
		
        $headers = array(
            'Authorization: key=' . GOOGLE_API_KEY,
            'Content-Type: application/json',
			
        );
        // Open connection
        $ch = curl_init();
 
        // Set the url, number of POST vars, POST data
        curl_setopt($ch, CURLOPT_URL, $url);
 
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
 
        // Disabling SSL Certificate support temporarly
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
 
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($fields));
		echo 'json request = ' . json_encode($fields);
        // Execute post
        $result = curl_exec($ch);
        if ($result === FALSE) {
            die('Curl failed: ' . curl_error($ch));
        }
		
        // Close connection
        curl_close($ch);
        echo 'GCM result = '.$result;
    }

step 6) After you have runed the app click  "Regiseter device" in mobile app which will show alert messages and finally you should get
	"Successfully Registered" message which means client app works.

step 7) call function in step 6 with regID stored in database with any message you want to send.You will get notification on you mobile device.
	(Again make sure internet is working in you mobile device.)
	Its working !!!!
