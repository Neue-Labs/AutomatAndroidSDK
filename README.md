# AutomatAndroidSDK
Android SDK for Automat by Neue LABS - BETA

Neue labs presents AUTOMAT, a small and flexible platform for building personal IoT. AUTOMAT is built to boost creativity and create real products for professional creatives in agencies, studios, innovation & R&D departments. Use the same hardware and software in both prototyping and industrialisation to cut costs dramatically and shorten time to market. Neue labs AUTOMAT quite simply makes great ideas happen.

Use this SDK to develop your own Android apps connecting via Bluetooth Low Energy to AUTOMAT hardware.

IMPORTANT! This is an early beta release of the SDK meaning that it lacks access to much of the functionality available on the devices. There can also be significant changes of the SDK in the future.

Currently, accelerometer data from a connected device can be read out.

## Installation

To add this to your project in Android Studio, do the following:
- Goto 'File' -> 'New' and select 'New Module'.
- Select 'Import .JAR/AAR Package', locate the 'neuelabsautomat-release.aar' and choose it for the import.
- In the 'build.gradle' file for the project, app level (Module: app), under 'dependencies', add: compile project(':neuelabsautomat-release')
- Sync gradle
- In the Android Manifest file add the following lines:
    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
- Also in the Android Manifest, in the <application> tag, add the following:
    <service android:name="com.neuelabs.neuelabsautomat.AutomatConnectionManager" android:enabled="true"/>

## Usage
First you need to make sure the user has granted permission for location access.
Second, you need to bind to the AutomatConnectionManager service. Binding to the service is done by:
```java
Intent gattServiceIntent = new Intent(this, AutomatConnectionManager.class);
bindService(gattServiceIntent, mServiceConnection, BIND_AUTO_CREATE);
...
private final ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder service) {
            mConnectionManager = ((AutomatConnectionManager.LocalBinder) service).getService();
            if (!connectionManager.initialize()) {
                finish();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {

        }
    };

```
As can be seen above, when the binding is complete and the service is connected, you call initialize on the connectionManager.

To actually start interacting with Bluetooth Low Energy devices you need to start scanning for them. This can be done in different ways depending on which Android SDK version is used.
Scan for a device, and if it has the name 'AUTOMAT', it's an Automat device and you can now connect to it in the following way:

```java
AutomatDevice automatDevice = connectionManager.getAutomatDevice(device.getAddress());
AutomatBaseboard board = (AutomatBaseboard) automatDevice;
mBaseboard = board;
board.setConnectionHandler(connHandler);
board.connect();
```
The connection handler that needs to be set on the AutomatBaseboard before connecting can look like the following:
```java
private AutomatConnectionHandler connHandler = new AutomatConnectionHandler() {
        @Override
        public void didConnectAutomatDevice() {
           Log.d(TAG, "Did connect Automat device");
            if (mBaseboard.getConnectionState() == AutomatDevice.AutomatConnectionState.CONNECTED) {
                mBaseboard.setMotionSensorODR(AutomatBaseboard.MotionSensorODRValue.NLAMotionLowPowerMode13Hz);
                mBaseboard.registerAutomatMotionHandler(motionHandler);
            }


        }

        @Override
        public void didDisconnectAutomatDevice(int i) {

        }

        @Override
        public void bluetoothGattFail(int i, String s) {

        }

        @Override
        public void bluetoothStatusChange(int i, String s) {

        }
    };
```
In the above example, when an Automat device is connected, we check to see if it's our mBaseboard that is connected. 
If so, set the output data rate, or ODR for the motion sensor to a value of choice and register a handler for receiving accelerometer data from the connected device.
A simple implementation of such a handler can be done as:
```java
private AutomatMotionHandler motionHandler = new AutomatMotionHandler() {
        @Override
        public void didReceiveMotionData(AutomatMotionData automatMotionData) {
            Log.d(TAG,"Did receive motion data: " + automatMotionData.getAcceleration().getX());
        }
    };
```

Check out [this project](https://github.com/Neue-Labs/Automat-Android-Example), to see a very simple demo project that displays the use of the above examples.





