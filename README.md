# AndroidBasics
Android Related Questions


<h1>Broadcast Receiver </h1>

Broadcast in android is the system-wide events that can occur when the device starts, when a message is received on the device or when incoming calls are received, or when a device goes to airplane mode, etc. Broadcast Receivers are used to respond to these system-wide events. Broadcast Receivers allow us to register for the system and application events, and when that event happens, then the register receivers get notified. There are mainly two types of Broadcast Receivers:

<h3> Static Broadcast Receivers: </h3>
These types of Receivers are declared in the manifest file and works even if the app is closed.

<h3> Dynamic Broadcast Receivers: </h3>
These types of receivers work only if the app is active or minimized. <br>
<br>
Intent - <br>
android.intent.action.BATTERY_LOW :	Indicates low battery condition on the device. <br>
android.intent.action.BOOT_COMPLETED : This is broadcast once after the system has finished booting <br>
android.intent.action.CALL :  To perform a call to someone specified by the data <br>
android.intent.action.DATE_CHANGED : Indicates that the date has changed <br>
android.intent.action.REBOOT	: Indicates that the device has been a reboot <br>
android.net.conn.CONNECTIVITY_CHANGE : The mobile network or wifi connection is changed(or reset) <br>
android.intent.ACTION_AIRPLANE_MODE_CHANGED	: This indicates that airplane mode has been switched on or off. <br>

<h3>Creating the Broadcast Receiver: </h3>

```
class AirplaneModeChangeReceiver:BroadcastReceiver() {

       override fun onReceive(context: Context?, intent: Intent?) {

            // logic of the code needs to be written here

      }

}
```

Registering a BroadcastReceiver:
```
IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED).also {
                     // receiver is the broadcast receiver that we have registered
                    // and it is the intent filter that we have created
                    registerReceiver(receiver,it)
      }
```

