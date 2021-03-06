# PART 4: Web-based HMI (Human-Machine Interface)

In this lab you will create a web-based, Human-Machine Interface (HMI) to monitor and control your virtual IoT device and its connected virtual sensors. Specifically, you will spin up an EC2 instance and install Node-RED (https://nodered.org/) and its **node-red-dashboard** module (https://flows.nodered.org/node/node-red-dashboard) to create a real-time dashboard to monitor and control your IoT Devices.

### Architecture

   ![Architecture](images/architecture-web-based-hmi-virtdev.png)


### 1. Launch EC2 Instance running Amazon Linux 2

   - Login to AWS console
   - Go to **Services/EC2** and select **Instances**
   - Press **Launch Instance** button


   ![Launch EC2 Instance](images/launch-ec2.png)

   - Choose the **Amazon Linux 2 - 64-bit** AMI and press the **Select** button


   ![Amazon Linux](images/amazon-linux.png)

   - Choose **General purpose, t2 micro** instance type and press the **Review and Launch** button


   ![T2 Micro](images/t2-micro.png)

   - Select **Edit security group**


   ![Edit Security Group](images/edit-security-group.png)

   - Press **Add Rule** button


   ![Add Security Group Rule](images/add-security-group-rule.png)

   - Select **Type=HTTP**, **Protocol=TCP**, **Port=80**, **Source=0.0.0.0, ::0**
   - Press **Review and Launch** button


   **NOTE: In a production environment you would typically create a more constrained Source for your security group rules**


   ![HTTP Security Group Rule](images/http-security-group-rule.png)

   - Select **Create a new key pair**
   - Enter a keypair name of **node-red**
   - Press **Download Key Pair** button


   ![Create Keypair](images/create-keypair.png)

   - Press **Launch Instances** button


   ![Launch Instance](images/launch-instances.png)

   - Periodically refresh the web page until EC2 instance indicates its status is running (i.e. 2/2 checks passed)
   - Take note of the public IP address. You will use this for your SSH hostname to access below. You will also use this for the URL when accessing your HMI dashboard


   ![Launch Instance](images/ec2-instance-running.png)

### 2. SSH onto the EC2 Instance

   - Launch the **Google Chrome Secure Shell App**
   - Enter a username of **ec2-user**, the public IP address of your EC2 instance and enter port **22**
   - Press **Import...** and import the **node-red.pem** file you downloaded when creating your EC2 instance.
   - Select the **node-red.pem** file when prompted.


   ![SSH to EC2](images/ssh-ec2.png)
   ![Select node-red.pem](images/node-red-pem.png)

   - Select the **node-red.pem** Identity you just imported (NOTE: If it doesn't show up in the selection list you may need to wait longer for the .pem file to upload or you many need to refresh the page on your web browser)
   - Press the **[ENTER] Connect** button.


   ![Connect via SSH](images/select-node-red-pem.png)

### 3. Install Node Red

   [ec2-user@ip-172-31-29-44 ~]$ **wget https://nodejs.org/dist/v10.18.1/node-v10.18.1-linux-x64.tar.gz**<br>
   [ec2-user@ip-172-31-29-44 ~]$  **sudo su**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **cd /usr/local**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **tar xvfz /home/ec2-user/node-v10.18.1-linux-x64.tar.gz**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **cd /usr/sbin**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **ln -s /usr/local/node-v10.18.1-linux-x64/bin/node**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **ln -s /usr/local/node-v10.18.1-linux-x64/bin/npm**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **exit**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **mkdir ~/iot-dashboard**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **cd ~/iot-dashboard**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **npm init** (press enter whenever prompted to accept default value)<br>
   [ec2-user@ip-172-31-29-44 ~]$ **npm install -s node-red**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **npm install -s node-red-dashboard**<br>
   [ec2-user@ip-172-31-29-44 ~]$ **sudo node node_modules/node-red/red.js --port 80**<br>


   ![Install Node Red](images/install-node-red.png)

### 4. Create HMI Dashboard to send commands to your virtual IoT device

   - Open another tab in Chrome
   - Enter a URL of **http://YOUR-EC2-INSTANCE-PUBLIC-IP-ADDRESS**
   - Drag and Drop a buttons from the pallet on the left onto the dashboard


   ![Add Buttons](images/add-button.png)

   - Edit the button by Double-clicking it
   - Press the pencil button to **Add new ui_group...**


   ![Add UI Group](images/add-ui-group.png)

   - Press the pencil button to **Add new ui_tab...**


   ![Add UI Tab](images/add-ui-tab.png)

   - Enter a **Name=IoT Workshop** and
   - Press the **Add** button


   ![Create UI Tab](images/create-ui-tab.png)

   - Enter a **Name=Default** and
   - Press the **Add** button


   ![Create UI Group](images/create-ui-group.png)

   - Edit the button you just dropped onto the dashboard by Double-clicking it
   - Select a Group of **[IoT Workshop] Default**
   - Enter a Label of **Beep**
   - Enter a Topic of **mything/beep** and
   - Press the **Done** button


   ![Create Beep Button](images/create-beep-button.png)

   - Drag and Drop **mqtt out** from the pallet on the left onto the dashboard


   ![Add MQTT](images/add-mqtt.png)

   - Edit **mqtt** by Double-clicking it
   - Press the pencil button to **Add new mqtt-broker...**


   ![Add MQTT Broker](images/add-new-mqtt.png)

   - Enter a Name of **iot-workshop**
   - Enter **Server** with the value of your MQTT Endpoint<br>
*(NOTE: This value may be obtained from Iot Core/Settings and referring to the Endpoint field)*
   - Enter a Port of **8883**
   - Enter a Client ID of **node-red**
   - Select **Enable secure (SSL/TLS) certificate**
   - Press the pencil button to **Add new tls-config...**


   ![Add TLS Config](images/create-new-tls.png)

   - Press Certificate **Upload** button and select **cert.pem**<br>
*NOTE: This is the cert.pem file you downloaded when registering your mything IoT device*
   - Press Private Key **Upload** button and select **private.pem**<br>
*NOTE: This is the private.pem file you downloaded when registering your mything IoT device*
   - Press CA Certificate **Upload** button and select **rootCA.pem**<br>
*NOTE: This is the rootCA.pem file you downloaded when registering your mything IoT device*


   ![Upload TLS Certs](images/upload-tls-certs.png)
   ![Upload Cert](images/upload-cert.png)
   ![Upload Private Key](images/upload-private.png)
   ![Upload Root CA](images/upload-rootca.png)

   - Press the **Add** button


   ![Update Certs](images/update-cert.png)

   - Press **Add** button


   ![Create MQTT Config](images/create-mqtt-config.png)

   - Press **Done** button


   ![Created MQTT Config](images/created-mqtt.png)

   - Connect Beep Button to MQTT


   ![Created MQTT Config](images/connect-mqtt.png)

   - Deploy your IoT Dashboard by pressing the **Deploy** button


   ![Created MQTT Config](images/deploy-dashboard1.png)

   - Open a new browser tab to view your dashboard in
   - Enter a URL of **http://YOUR-EC2-INSTANCE-PUBLIC-IP-ADDRESS/ui**
   - Press the **Beep** button and your virtual Device will beep


   ![Created MQTT Config](images/test-dashboard1.png)

### 5. Modify HMI Dashboard to display IoT telemetry data from your virtual device

   - Drag and Drop **mqtt in** from the pallet on the left onto the dashboard


   ![Add MQTT](images/add-mqtt-in.png)

   - Edit **mqtt** you just dropped onto your dashboard by double-clicking it
   - Select a Server of **iot-workshop**
   - Enter a Topic of **mything/temperature**
   - Select a QoS of **1**
   - Press the **Done** button


   ![Edit MQTT](images/edit-mqtt-in.png)

   - Drag and Drop **Function** from the pallet on the left onto the dashboard


   ![Add Function](images/add-function.png)

   - Edit **Function** by double-clicking it
   - Enter a Name of **Extract Value**
   - Enter the following Function code:
   <pre>
   var payload = JSON.parse(msg.payload);
   var newmsg  = { payload: payload.value };

   return newmsg;
   </pre>
   - Press the **Done** button


   ![Edit Function](images/edit-function.png)

   - Drag and Drop **Chart** from the pallet on the left onto the dashboard


   ![Add Chart](images/add-chart.png)

   - Edit **Chart** by double-clicking it
   - Select a Group of **[IoT Workshop] Default**
   - Enter a Label of **Water Temperature**
   - Press the **Done** button


   ![Edit Chart](images/edit-chart.png)

   - Drag and Drop **Gauge** from the pallet on the left onto the dashboard


   ![Add Gauge](images/add-gauge.png)

   - Edit **Gauge** by double-clicking it
   - Select a Group of **[IoT Workshop] Default**
   - Enter a Label of **Water Temperature**
   - Enter Units equal to **F**
   - Enter a min of **0**
   - Enter a max of **100**
   - Press the **Done** button


   ![Edit Gauge](images/edit-gauge.png)

   - Connect MQTT to Extract Value Function
   - Connect Extract Value Function to the Water Temperture Chart
   - Connect Extract Value Function to the Water Temperture Gauge


   ![Connect](images/connect-mqtt-func-chart-gauge.png)

   - Depoly your IoT Dashboard by pressing the **Depoly** button


   ![Created MQTT Config](images/deploy-dashboard2.png)

   - Go back to your browser tab with the IoT Dashboard in it
   - Move the slider of your virtual IoT device and see the chart and gauge on your dashboard change


   ![Created MQTT Config](images/test-dashboard2.png)

   **BONUS POINTS: Modify your dashboard to have Amazon colors as shown above**

# Continue Workshop

[Part 5 - Thing Shadows](./Workshop5-Virtual-thing-shadow.md)