# PART 2: AWS IoT Device Registration

In this lab you will convert your Raspberry Pi program, which can control and monitor its attached Grove Pi+ sensors, to securely connect as a **"Thing"** to the AWS IoT cloud. We will also enhance your application to function as a water dispensing station. Specifically, we will monitor the proximity sensor to detect water bottles being placed into and removed from the water dispenser. When we detect that a water bottle is in the water dispenser, we'll indiate that this event has been detected on the LCD display attached to the Raspberry Pi. When the water bottle is detected for longer than a configurable number of seconds (e.g. 2 seconds) we will trigger an event indicating that the water bottle is being filled. When the water bottle is removed, a proximity event indicating this will trigger it to indicate that the filling is completed. This will result in the application emitting a **beep** on the buzzer and updating the LCD display attached to the Raspberry Pi. The LCD display will indicate the accumulated total number of bottles filled as well as the current state - **Idle**, **Detected** or **Filling**. Additionally, IoT events will be published to AWS IoT Core via secure MQTT messages. These events include the state of the water dispenser as it changes from **Idle** to **Detected** to **Filling** as well as an event indicating that a water bottle has been filled. This **bottle filled** event will include data such as the time it was filled, the duration to fill it and the amount of water dispensed.

### Architecture


   ![Select Region](images/architecture-device-registration.png)

### 1. Register your device to AWS IoT Core
   - Login to AWS console and ensure your region is **US East (N. Virginia)**


   ![Select Region](images/select-region.png)

   - Go to **Services/IoT Core** by entering **IoT Core** in the Find Services field and pressing **Enter**


   ![IoT Core](images/select-iot-core.png)

   - If this is the first time you've used **IoT Core** you may be presented with a **Get Started** dialog
   - Press the **Get started** button to continue


   ![IoT Core](images/iot-get-started.png)

   - Select **Manage/Things**
   - Press **Register a thing** button


   ![Register Thing](images/register-thing.png)

   - Press **Create a single thing** button
   

   ![Create Thing](images/create-thing.png)
   
   - Enter **Name=mything** and press **Next** button
   

   ![Create Thing](images/create-thing-next.png)

### 2. Create a certificate for your IoT thing
   - Press **Create certificate** button


   ![Create Certificate](images/create-certificate.png)

   - Download **A certificate for this thing** and save it as **Downloads/certs/cert.pem** on your laptop
   - Download **public key** and save it as **Downloads/certs/public.pem** on your laptop
   - Download **private key** and save it as **Downloads/certs/private.pem** on your laptop
   - Download **A root CA for AWS IoT** save it as **Downloads/certs/rootCA.pem** on your laptop


   ![Download Certificate](images/download-certs.png)
---
   ![Download Root Certificate - Part2](images/get-rootca2.png)
---
   - Close the CA Certificates tab pane once you've dowloaded your root CA cert file.
   - Press **Done** button *(make sure you've downloaded your certs as shown above before you do this)*


   ![Download Certificate](images/download-certs2.png)


### 3. Create an IoT Policy for your Thing
   - Select **Secure/Policies**
   - Press "Create a policy" button


   ![Create Policy](images/create-policy1.png)
   
   - Enter **Name=mything-policy**
   - Enter **Action=iot:\***
   - Enter **Resource ARN=\***
   - Select **Effect=Allow**
   - Press **Create** button<br>
     *NOTE: Typically you would make a more constrained policy in a production environment*


   ![Create Policy](images/create-policy2.png)
---
   ![Create Policy](images/create-policy-done.png)

### 4. Attach your IoT Policy to the device certificate
   - Select **Secure/Certificates**
   - Select **...** pulldown menu of your certificate, and select **Attach policy**


   ![Create Policy](images/attach-policy1.png)

   - select your policy **mything-policy**
   - press **Attach** button


   ![Create Policy](images/attach-policy2.png)
   ![Create Policy](images/attach-policy-done.png)

### 5. Attach your certificate to your Thing and activate the certificate
   - Select **Secure/Certificates**
   - Select **...** pulldown menu of your certificate, and select **Attach thing**


   ![Attach Thing](images/attach-thing1.png)

   - Select your thing **mything**
   - Press **Attach** button


   ![Attach Thing](images/attach-thing2.png)

   - We'll now activate your certificate.
   - Select **...** pulldown menu of your certificate, and select **Activate**


   ![Activate Certificate](images/activate-cert.png)

### 6. Locate your IoT Custom Endpoint value
   - Select **Settings** and take note of your **Custom Endpoint** (your value may differ from the example shown below).<br>
   

   **NOTE: Save this value in your notes for the future. It will be used when you configure the host which your IoT code will connect to below**


   ![Custom Endpoint](images/custom-endpoint.png)

### 7. Secure copy your previously saved **mything** IoT certificates onto your Raspberry Pi

   - Launch the **Google Chrome Secure Shell App** in another tab to start a SFTP session
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi, enter port **80** and press the **SFTP** button.

   
   **NOTE: The password to use when logging in is written on your Raspberry Pi case. Also note that the IP address assigned to your Raspberry Pi may differ from the example shown in the screen capture below.**


   ![SFTP Raspberry Pi](images/sftp-raspberry-pi.png)

  nasftp ./ > **cd ~/Development/iot-workshop**<br>
  nasftp /home/pi/Development/iot-workshop/ > **mkdir certs**<br>
  nasftp /home/pi/Development/iot-workshop/ > **cd certs**<br>
  nasftp /home/pi/Development/iot-workshop/certs/ > **put**<br>

   ![SFTP Raspberry Pi](images/sftp-raspberry-pi-put-file.png)

  - Select your **mything** IoT certs that you previously stored in **Downloads/certs**

**IMPORTANT: Make sure the names of your files are EXACTLY as shown below. If not you should rename the files to match those exact names or you will not be able to connect your thing successfully to AWS IoT**

   ![SFTP Raspberry Pi](images/sftp-raspberry-pi-save-put.png)

### 8. SSH onto Raspberry PI using Chrome SSH App

   - Launch the **Google Chrome Secure Shell App** in another tab to start a SSH session
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi, enter port **80** and press the **[ENTER] Connect** button.

   
   **NOTE: The password to use when logging in is written on your Raspberry Pi case. Also note that the IP address assigned to your Raspberry Pi may differ from the example shown in the screen capture below.**


   ![SSH to Raspberry Pi](images/ssh-to-raspberry-pi.png)

### 9. Update your application to connect to AWS IoT Core as your **mything** you previously created using the AWS Console

   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop**<br>
   pi@raspberrypi:~ $ **npm install -s aws-iot-device-sdk**<br>

   ![npm init](images/aws-iot-device-sdk.png)
   - Use your favorite editor to modify the test program to connect to AWS IoT Core.<br><br>

   pi@raspberrypi:~ $ **nano index.js** *(press control-X to exit and press Y to save the file)*<br>
   

   ![npm init](images/nano-edit2.png)

   The highlighted lines in the code below are the lines added to securely connect to AWS IoT Core as well as the additional water dispensing functionality. This was provided for educational purposes and you may find it easier to simply cut-and-paste the entire code.

   **IMPORTANT: Replace the host: value with the Custom Endpoint value you took note of earlier**<br>
   **HINT: If you forgot your Custom Endpoint value you can get it from Iot Core/Settings**<br>
   **HINT: When using nano use shift and arrow keys to select text and control-K to delete text**<br>
   **HINT: Right mouse button may be used to copy and paste when using Google Chrome SSH**

<pre>
<b style="color:red">const AWSIoT  = require('aws-iot-device-sdk');
const Display = require('./display');</b>
const GrovePi = require('grovepi').GrovePi;
const Board   = GrovePi.board;
const Sensors = GrovePi.sensors;

const red    = new Sensors.DigitalOutput(2);
const blue   = new Sensors.DigitalOutput(3);
const buzzer = new Sensors.DigitalOutput(8);
const prox   = new Sensors.UltrasonicDigital(7);
const angle  = new Sensors.RotaryAnalog(0);
<b style="color:red">
const DETECT_PROXIMITY = 5;
const DETECT_DURATION  = 2;    // 2 secs
const DISPENSE_RATE    = 200;  // 200 ml/sec
const PROCESS_INTERVAL = 1000; // 1000 msecs

const IDLE     = 0;
const DETECTED = 1;
const FILLING  = 2;

let device      = null;
let state       = IDLE; // IDLE, DETECTED or FILLING (Default = IDLE)
let proximity   = 200;  // 0 = nearest, 200 = farthest (Default = 200)
let temperature = 70;   // Fahrenheit (Default =70)
let time        = Date.now();
let total       = { filled: 0, volume: 0 };
</b>
function main() {
  const board = new Board({
    debug: true,
    onError: onGroveError,
    onInit: onGroveInit
  });

  board.init();
  <b style="color:red">
  device = AWSIoT.device({
    keyPath:  './certs/private.pem',
    certPath: './certs/cert.pem',
    caPath:   './certs/rootCA.pem',
    clientId: 'mything',
    region:   'us-east-1',
    protocol: 'mqtts',
    // IMPORTANT: REPLACE THE HOST VALUE BELOW WITH YOUR CUSTOM ENDPOINT VALUE
    host:     'a26erfmc9c0soi-ats.iot.us-east-1.amazonaws.com',
    debug:    'true'
  });

  device.on('connect', onConnect);
  device.on('message', onMessage);</b>
}
<b style="color:red">
function onConnect() {
  console.log('Connected to AWS IoT Core');
  device.subscribe('mything/beep');
  device.subscribe('mything/red/#');
  device.subscribe('mything/blue/#');
  device.publish('mything/dispenser', '{ "state": "idle", "time": ' + Date.now() + ' }');
}

function onMessage(topic, buffer) {
  if (topic === 'mything/beep') {
    beep();
  }
  else if (topic === 'mything/red/on') {
    red.turnOn();
  }
  else if (topic === 'mything/red/off') {
    red.turnOff();
  }
  else if (topic === 'mything/blue/on') {
    blue.turnOn();
  }
  else if (topic === 'mything/blue/off') {
    blue.turnOff();
  }
}
</b>
function onGroveError(err) {
  console.log('ERROR: ' + JSON.stringify(err));
}

function onGroveInit(res) {
  console.log('GrovePi initialized');
  monitorAngle();
  monitorProximity();
  setInterval(process, PROCESS_INTERVAL);
}

function beep() {
  console.log('Beep');
  buzzer.turnOn();
  setTimeout(() => { buzzer.turnOff(); }, 1000);
}

function monitorAngle() {
  console.log('Monitor Angle ...');

  angle.on('change', res => {
    <b style="color:red">temperature = res;
    device.publish('mything/temperature', '{ "value":' + temperature + ' }');</b>
  });

  angle.watch(1000);
}

function monitorProximity() {
  console.log('Monitor Proximity ...');

  prox.on('change', res => {
    <b style="color:red">proximity = res;</b>
  });

  prox.watch(1000);
}
<b style="color:red">
function process() {
  let now       = Date.now();
  let duration  = Math.floor((now - time) / 1000);

  if (state === IDLE) {
    if (proximity > DETECT_PROXIMITY) {
      display('Idle', proximity, Display.GREEN);
    }
    else {
      state = DETECTED;
      time  = now;
      display('Detected', proximity, Display.BLUE, 0);
      device.publish('mything/dispenser', '{ "state":"detected" , "time":' + now + ' }');
    }
  }
  else if (state === DETECTED) {
    if (proximity > DETECT_PROXIMITY) {
      state = IDLE;
      display('Idle', proximity, Display.GREEN);
      device.publish('mything/dispenser', '{ "state":"idle", "time":' + now + ' }');
    }
    else if (duration >= DETECT_DURATION) {
      state = FILLING;
      time  = now;
      display('Filling', proximity, Display.RED, duration);
      device.publish('mything/dispenser', '{ "state":"filling", "time":' + now + ' }');
    }
    else {
      display('Detected', proximity, Display.BLUE, duration);
    }
  }
  else if (state == FILLING) {
    if (proximity > DETECT_PROXIMITY) {
      let dispensed = duration * DISPENSE_RATE;

      state = IDLE;
      total.filled += 1;
      total.volume += dispensed;
      beep();

      display('Idle', proximity, Display.GREEN);
      device.publish('mything/dispenser', '{ "state":"idle", "time":' + now + ' }');
      device.publish('mything/bottle', '{ "total":{ "filled":' + total.filled + ', "volume":' + total.volume + '},' +
                                          '"time":' + now + ', "duration":' + now + ', "volume":' + dispensed + ' }');
    }
    else {
      display('Filling', proximity, Display.RED, duration);
    }
  }
}

function display(state, proximity, color, duration) {
  if (duration) {
    Display.setText('Filled: ' + total.filled + '\n' + state + '[' + proximity + ']: ' + duration + 's', 1, color);
  }
  else {
    Display.setText('Filled: ' + total.filled + '\n' + state + '[' + proximity + ']', 1, color);
  }
}
</b>
main();
</pre>

   - Use your favorite editor to create display.js to control the LCD display connected to your Raspberry Pi.<br><br>

   pi@raspberrypi:~ $ **nano display.js** *(press control-X to exit and press Y to save the file)*<br>
   

   ![npm init](images/nano-edit5.png)

   **HINT: Right mouse button may be used to copy and paste when using Google Chrome SSH**

<pre>
<b style="color:red">const i2cBus = require('i2c-bus');
const sleep  = require('sleep');

const DISPLAY_RGB_ADDR  = 0x62;
const DISPLAY_TEXT_ADDR = 0x3e;

const RED   = { red: 255, green: 0,   blue: 0   };
const GREEN = { red: 0,   green: 255, blue: 0   };
const BLUE  = { red: 0,   green: 0,   blue: 255 };

function setRGB(i2c, r, g, b) {
  i2c.writeByteSync(DISPLAY_RGB_ADDR, 0, 0);
  i2c.writeByteSync(DISPLAY_RGB_ADDR, 1, 0);
  i2c.writeByteSync(DISPLAY_RGB_ADDR, 0x08, 0xaa);
  i2c.writeByteSync(DISPLAY_RGB_ADDR, 4, r);
  i2c.writeByteSync(DISPLAY_RGB_ADDR, 3, g);
  i2c.writeByteSync(DISPLAY_RGB_ADDR, 2, b);
}

function textCommand(i2c, cmd) {
  i2c.writeByteSync(DISPLAY_TEXT_ADDR, 0x80, cmd);
}

function sendText(i2c, text) {
  textCommand(i2c, 0x01); // clear display

  sleep.usleep(50000);
  textCommand(i2c, 0x08 | 0x04); // display on, no cursor
  textCommand(i2c, 0x28);        // 2 lines
  sleep.usleep(50000);

  let length = text.length;
  let count  = 0;
  let row    = 0;

  for (let i = 0; i < length; i++) {
    if (text[i] === '\n' || count === 16) {
      count = 0;
      row++;

      if (row === 2) {
        break;
      }

      textCommand(i2c, 0xc0);

      if (text[i] === '\n') {
        continue;
      }
    }

    i2c.writeByteSync(DISPLAY_TEXT_ADDR, 0x40, text[i].charCodeAt(0));
    count++;
  }
}

function setText(text, port, color) {
  try {
    let i2c = i2cBus.openSync(port);

    console.log(text);
    sendText(i2c, text);
    setRGB(i2c, color.red, color.green, color.blue);
    i2c.closeSync();
  }
  catch (err) {
    console.log('ERROR: ' + JSON.stringify(err));
  }
}

exports.setText = setText;

exports.RED   = RED;
exports.GREEN = GREEN;
exports.BLUE  = BLUE;
</b>
</pre>


### 10. Show connectivity to AWS IoT Core and test your **mything**
   - Login to AWS console
   - Go to **Services/IoT Core**
   - Select **Test**
   - Subscribe **topic=mything/#** and press **Subscribe to topic** button


   ![MyThing Subscribe](images/mything-subscribe.png)

   - Once subscribed, you'll see a message "Hello from AWS IoT console"


   ![MyThing Subscribed](images/mything-subscribed.png)

### 11. Connect your Raspberry Pi as **mything** to AWS IoT Core

   - Go back to your tab where you're SSH-ed onto your Raspberry Pi and run the program<br>
   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop**<br>
   pi@raspberrypi:~ $ **node index.js**<br>


   ![Connect to AWS IoT Core](images/connect-to-AWS.png)

   - The application indicates "Connected to AWS IoT Core"
   - The IoT Test page shows an incoming **ONLINE** message from the Raspberry Pi


   ![Connect to AWS IoT Core](images/connected-publications.png)

   - Turn angle sensor on Raspberry Pi and simulate incoming **temperature** messages


   ![Angle Tests Publications](images/angle-test-pubs.png)

   - Publish a message to beep the buzzer on the Raspberry Pi
   - Enter a Publish **topic** of **mything/beep**
   - Press **Publish to topic** button


   ![MyThing Publish](images/publish-beep.png)

   - Publish a message to turn on the red LED attached to the Raspberry Pi
   - Enter a Publish **topic** of **mything/red/on**
   - Press the **Publish to topic** button


   ![MyThing Publish](images/publish-red-on.png)

   - Publish a message to turn off the red LED attached to the Raspberry Pi
   - Enter a Publish **topic** of **mything/red/off**
   - Press the **Publish to topic** button


   ![MyThing Publish](images/publish-red-off.png)

   - Publish a message to turn on the blue LED attached to the Raspberry Pi
   - Enter a Publish **topic** of **mything/blue/on**
   - Press the **Publish to topic** button


   ![MyThing Publish](images/publish-blue-on.png)

   - Publish a message to turn off the blue LED attached to the Raspberry Pi
   - Enter a Publish **topic** of **mything/blue/off**
   - Press the **Publish to topic** button


   ![MyThing Publish](images/publish-blue-off.png)
