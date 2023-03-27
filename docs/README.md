# saiot-arduino
arduino board-package targeting Santagostino IoT devices

# Changelog
* v0.0.4: first official stable-release

# Prerequisite
* Santagotino IoT Boards
* Arduino 2.X IDE
# Installation
1. Open Arduino 2.X IDE
2. Open the menu File->Preferences
3. Copy the following link in "Additional boards manager URLs": https://raw.githubusercontent.com/AL2TECH/saiot-arduino/main/package_saiot_arduino_index.json
![Arduino preferences](imgs/preferences.PNG "Arduino preferences")
4. Open the Boards Manager: Tools->Board->Board Manager.
5. Search for "santagostino" board package and install the latest available version.

![Arduino boards manager](imgs/boards_manager.PNG "Arduino boards manager")

6. If installation has been successful, a new board package selection (i.e., SANTAGOSTINO IOT Boards) and a list of related boards (e.g., HPM (Heat Pump Monitor)) should appear in Tools->Board. Choose the proper target board when developing.

![SAIOT board selection](imgs/saiot_boards_selection.PNG "SAIOT board selection")

# Board setup/configuration
1. When the board is powered up for the first time o when has been flashed with low power mode enabled (details in dedicated section), the board must be forced to go into boot mode: 
    1. Press and hold both reset and user button.

    ![Buttons](imgs/buttons.PNG "Buttons")

    2. Release the reset button.
    3. Release the user button.
    4. Now the boards should appear as COM device.
2. In Tools->"USB CDC On Boot" always set to Enabled: it redirects serial to USB.

![USB CDC Enabled](imgs/cdc_enabled.PNG "USB CDC Enabled")

3. Choose the right COM port.

# API Documentation: SAIOT_Board class
* package version: 0.0.4
* This section describes the available methods and public variables in SAIOT_Board class.
## Public variables
## board_status_t status
Public variable holding last available board status. It values:
* BOARD_STATUS_OK: no error.
* BOARD_STATUS_WIFI_ERROR: error in wifi connection.
* BOARD_STATUS_NTP_ERROR: error in getting time using NTP.

## bool lowPowerEnable
Public variable for setting low power mode. Default: false.
When lowPowerEnable is true the low power mode is enabled: when no one task is in execution the board is put in deep sleep (i.e., the lowest power consumption mode available).
Otherwise when no one task is in execution a simple delay is called.
## long wifiConnTimeout
The maximum timeout seconds to attempt wifi connection (could be shorter if wifi ssid is not available during esp32 module async wifi scan). Default: 10 seconds.
## SAIOT_MQTT* mqtt
Pointer to a SAIOT_MQTT globally initialized class object.
## SAIOT_LED* led
Pointer to a SAIOT_LED globally initialized class object.
## SAIOT_BUTTON* button
Pointer to a SAIOT_BUTTON globally initialized class object.
## SAIOT_LSM6DSOXClass* imu
Pointer to a SAIOT_LSM6DSOXClass globally initialized class object.
## SAIOT_Expansion* expansion
Pointer to a SAIOT_Expansion globally initialized class object.
## Methods
## board_status_t begin(ssid, pwd, time_offset_sec)
Initializes the board by setting wifi ssid, wifi password, testing the connection and set the correct time using NTP (Network Time Protocol-Unix Epoch: seconds elapsed since 00:00:00 UTC). Attention: If you have set a time offset this time offset will be added to your epoch timestamp.
### Parameters
* const char* ssid: string containing the wifi ssid name.
* const char* pwd: string containing the wifi password.
* const long time_offset_sec: the local time offset in seconds to add to time get by NTP.
### Returns
* BOARD_STATUS_OK: success.
* BOARD_STATUS_WIFI_ERROR: error in wifi connection.
* BOARD_STATUS_NTP_ERROR: error in getting time using NTP.

## void addTasks(tasks, num_tasks)
Add an array of tasks. The max number of tasks handled is 10.
### Parameters
* SAIOT_Task tasks[] tasks: array of tasks, the maximum supported number of task is 10.
* const uint8_t num_tasks: the tasks array size, must be less or equal than 10.
## bool wifiConnect()
Connects to wifi. It uses wifiConnTimeout as timeout seconds to attempt wifi connection.
### Returns
* true: connection success.
* false: connection failed.
## bool wifiDisconnect()
Disconnects to wifi.
### Returns
* true: disconnection success.
* false: disconnection failed.
## bool isWifiConnected()
Check wifi connection.
### Returns
* true: wifi is connected.
* false: wifi is not connected.
## bool getTimeEpochFormat()
get the number of seconds since Unix epoch.
### Returns
* uint32_t: number of seconds since Unix epoch.
## loop()
Handles tasks execution and deep sleep/delay. This should be called as final statement inside Arduino void loop().
# API Documentation: SAIOT_Task class
* package version: 0.0.4
* This section describes the main available methods in SAIOT_Task class.

## Methods
## SAIOT_Task(task_id, interval_time,callback, [offset_time=0], [task_enabled=true])
Creates a task instance.
### Parameters
* const uint8_t task_id: it ranges [0,9] (max 10 tasks handled).Indicates also priority (lower the value higher the priority): between two tasks that need to run at the same time the one with higher priority takes precedence.
* const uint32_t interval_time: time period in seconds.
* task_callback callback: the callback containing task job (i.e., void callback(SAIOT_Task* task)).
* const uint32_t offset_time (optional): an offset time in seconds for delayed execution.
* const bool task_enabled (optional): indicates if enable or disable the task.

# API Documentation: SAIOT_MQTT class
* package version: 0.0.4
* This section describes the main available methods in SAIOT_MQTT class.
## Methods
## bool connect(client_id, username, pwd, server, port)
Connects to a mqtt server, providing client_id, username, password, server address and server port.
### Parameters
* const char* client_id: string containing the client ID.
* const char* username: string containing client username.
* const char* pwd: string containing client password.
* const char* server: string containing server address (the server url).
* const int port: the server port number.
### Returns
* true: connection success.
* false: connection failed.
 ## void disconnect()
 Disconnects from mqtt server.
 ### Returns
* true: disconnection success.
* false: disconnection failed.
## bool isConnected()
Checks mqtt connection.
### Returns
* true: mqtt is connected.
* false: mqtt is not connected.
##  bool publish(topic, payload)
Publishes a payload(string) in provided topic.
### Parameters
* const char* topic: the topic where to publish.
* const char* payload: the payload to publish. Max 256 bytes allowed.
### Returns
* true: publish success.
* false: publish failed.
## bool subscribe(topic, MQTT_CALLBACK_SIGNATURE)
Subscribes to provided topic.
### Parameters
* const char* topic: the topic to subscribe.
* MQTT_CALLBACK_SIGNATURE: the callback called when a message is received (i.e., void callback(char *topic, byte *payload, unsigned int length))
### Returns
* true: subscribe success.
* false: subscribe failed.
## bool unsubscribe(topic)
Unsubscribes to provided topic.
### Parameters
* const char* topic: the topic to unsubscribe.
### Returns
* true: unsubscribe success.
* false: unsubscribe failed.
## bool loop(timeout_msec)
Processes incoming messages until a timeout is elapsed. Returns true if client is still connected, false otherwise.
### Parameters
* const long timeout_msec: the amount of time in milliseconds for receive/process incoming messages on subscribed topic.
### Returns
* true: client still connected.
* false: client not connected.
## int getState()
Returns mqtt state.
### Returns
* MQTT_CONNECTION_TIMEOUT     -4
* MQTT_CONNECTION_LOST        -3
* MQTT_CONNECT_FAILED         -2
* MQTT_DISCONNECTED           -1
* MQTT_CONNECTED               0
* MQTT_CONNECT_BAD_PROTOCOL    1
* MQTT_CONNECT_BAD_CLIENT_ID   2
* MQTT_CONNECT_UNAVAILABLE     3
* MQTT_CONNECT_BAD_CREDENTIALS 4
* MQTT_CONNECT_UNAUTHORIZED    5

# API Documentation: SAIOT_LSM6DSOXClass class
* package version: 0.0.4
* The methods are the same provided in Arduino_LSM6DSOX official library, version 1.1.2 (ref: https://reference.arduino.cc/reference/en/libraries/arduino_lsm6dsox/).