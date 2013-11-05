One of the first goals that we had in mind when we picked up our Arduino Yún<a href="#1"><sup>[1]</sup></a><img src="http://o7.no/16BqnRc" width="5%;"style="vertical-align:middle;"/>  was to get it connected to <img src="http://dl.dropboxusercontent.com/u/980/Screenshots/smkx_e~bj7co.png" width="3%;"style="vertical-align:middle;"/>[Firebase](https://www.firebase.com/). Now that the Yún has been released, let's take a look back from where we've come.

Our feeble attempts with our Arduino <a href="#1"><sup>[1]</sup></a> and the Offical Ethernet Shield <img src="http://o7.no/1cAW7uz" width="5%;"style="vertical-align:middle"> <a href="#2"><sup>[2]</sup></a> we're soon thwarted by the Arduino's inability to https "possible due to the size and complexity of the SSL libraries." <sup><a href="#3">[3]</a></sup>.

People had offered up a bevy of [solutions](http://stackoverflow.com/questions/11138017/https-alternative-on-arduino)<sup> [[4]](#4)</sup> ranging from php proxy to some other encryption options. We ended up back to the internets asking our [own question](http://stackoverflow.com/questions/18005062/how-to-address-firebase-from-an-arduino/18117622#18117622) to see what we might find.

The same types of answers popped up, so we went with our own php script hosted on Heroku that just serves as a proxy, using [Firebase PHP Client](https://github.com/ktamas77/firebase-php).
```javascript
<?php
require_once 'firebaseLib.php';
$url = 'YOUR FIREBASE URL';
$token = 'YOUR FIREBASE TOKEN HERE';
$arduino_data = $_GET['arduino_data'];
// --- Set up your Firebase url endpoint here
$firebasePath = '/';
/// --- Making calls
$fb = new fireBase($url, $token);
$response = $fb->set($firebasePath, $rfidid);
sleep(2);
```
We included the library from github in order to facilitate communication and JSON mapping that Firebase needs.
Now that the Yún has Linux on-board, we can utilize new tools to help us communicate in real-time.
Our first few attempts involved trying to CURL from the Linux side of the board by communicating across the [Bridge library](http://arduino.cc/en/Reference/YunBridgeLibrary) using the Process function to run a shell command (CURL).
```bash
Process p;    // Create a process and call it "p"
p.runShellCommand("curl -k -X PATCH https://YOURDATABASE.firebaseio.com/.json -d '{ \"motion\" : \"Detected!\" }'");  

```
We needed to run the CURL command with the option -k (insecure) in order to make it work. [Libcurl](http://curl.haxx.se/libcurl/) that's installed on OpenWRT hasn't been compiled to support SSL, so we're vulnerable to a man-in-the-middle attack.

Out of the box, Arduino Yún has [Temboo](https://www.temboo.com/arduino) built-in. It's an awesome place to "Create, Make, Code the Internet of Everything". I whole-hearted agree. With a SDK with 100+ APIs written in 7 langauges.

One of their [Choreos](https://www.temboo.com/library/) from the [Utility library](https://www.temboo.com/library/Library/Utilities/) is the [HTTP library](https://www.temboo.com/library/Library/Utilities/HTTP/). It's a "bundle includes functionality for generating GET, POST, PUT, and DELETE HTTP requests."
We'll give this a crack to see if we can secure our data as we put it up into the cloud…
You'll need to include a TembooAccount.h to your Arduino Sketch to definte your AccountName, AppKeyName, and your AppKey.

```C#
#include <Bridge.h>  
#include <Temboo.h>
#include "TembooAccount.h" // contains Temboo account information, as described below
#include <TinkerKit.h>


String startString;
long hits = 0;
float F;
TKThermistor therm(I0);       // creating the object 'therm' that belongs to the 'TKThermistor' class
TKLightSensor ldr(I1);	//create the "ldr" object on port I1
char serverName[] = "yunyun.heroku.com"; //The important bit
int brightnessVal;
int numRuns = 1;   // execution count, so this doesn't run forever
int maxRuns = 10;   // maximum number of times the Choreo should be executed

void setup() {
  Serial.begin(9600);
  
  // For debugging, wait until a serial console is connected.
  delay(4000);
  while(!Serial);
  Bridge.begin();
}
void loop()
{
  if (numRuns <= maxRuns) {
    Serial.println("Running Put - Run #" + String(numRuns++));
      F = therm.readFahrenheit();  	// Reading the temperature in Fahrenheit degrees and store in the F variable
      int brightnessVal = ldr.read() ;   
    TembooChoreo PutChoreo;

    // invoke the Temboo client
    PutChoreo.begin();
    
    // set Temboo account credentials
    PutChoreo.setAccountName(TEMBOO_ACCOUNT);
    PutChoreo.setAppKeyName(TEMBOO_APP_KEY_NAME);
    PutChoreo.setAppKey(TEMBOO_APP_KEY);
    
    // set choreo inputs
    PutChoreo.addInput("RequestBody", "{ F : \"Jack\", \"last\": \"Spasadrrow\" }");
    PutChoreo.addInput("RequestHeaders", "{ \"Content-Type\":\"application/json\"}");
    PutChoreo.addInput("URL", "https://yun.firebaseio.com/Names/.json");
    // or 
    // identify choreo to run
    PutChoreo.setChoreo("/Library/Utilities/HTTP/Put");
    
    // run choreo; when results are available, print them to serial
    PutChoreo.run();
  delay(50); // Poll every 50ms

    while(PutChoreo.available()) {
      char c = PutChoreo.read();
      Serial.print(c);
    }
    PutChoreo.close();

  }

  Serial.println("Waiting...");
  delay(30000); // wait 30 seconds between Put calls
}
```


1. ^ Arduino Yún ([Reference](http://arduino.cc/en/Main/ArduinoEthernetShield), [Purchase](https://www.sparkfun.com/products/9026))
2. ^ Ethernet Shield ([Reference](http://arduino.cc/en/Main/ArduinoEthernetShield), [Purchase](https://www.sparkfun.com/products/9026))
3. ^ ["HTTPS interfacing"](http://forum.arduino.cc/index.php/topic,13134.0.html) by [Adumas](http://forum.arduino.cc/index.php?PHPSESSID=ecavtkhrlkkjs8kv3g8s5uch31&action=profile;u=9095) on [Arduino Forums](http://forum.arduino.cc/) (June 05, 2009).
4. ^ ["Encryption - HTTPS alternative on Arduino"](http://stackoverflow.com/questions/11138017/https-alternative-on-arduino) on [Stack Overflow](http://stackoverflow.com) (Jun 21, 2012) 
