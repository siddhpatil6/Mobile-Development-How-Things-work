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

<h3> Registering a BroadcastReceiver:
       
```
IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED).also {
                     // receiver is the broadcast receiver that we have registered
                    // and it is the intent filter that we have created
                    registerReceiver(receiver,it)
      }
```


<h1> Retrofit Interceptors for Beginners </h1>
interceptors are a powerful concept that can monitor, rewrite, authenticate, cache and retry the API call. So basically, when we request API call, we can monitor the call or perform some tasks or set of interceptors in one call is also allowed. In a nutshell, Interceptors function very similar to airport security personnel during the security check process. They checked our boarding pass, stamped it, and then let us pass. <br>

<h2> Interceptor Types </h2>
Interceptor Types

<h3> 1. Application Interceptors </h3>
Interceptors added between the Application Code (our written code) and the OkHttp Core Library are referred to as application interceptors. These are the interceptors that we add with addInterceptor(). <br>

<h3> 2. Network Interceptors </h3>
Interceptors on the network: These are interceptors placed between the OkHttp Core Library and the server. These can be added to OkHttpClient by using the addNetworkInterceptor(). <br>

<h2> Few general interceptors use-cases </h2>

<h3> 1. Logging interceptor </h3>
In Retrofit 2, all network operations are performed via OkHttp library. OkHttp provides HttpLoggingInterceptorwhich logs HTTP request and response data. <br>

HttpLoggingInterceptor has 4 levels of logging: <br>
<br>
BASIC – Logs request and response lines.<br>
<br>
BODY – Logs request and response lines and their respective headers and bodies (if present). This is the only log level where we will get the response body data. <br>
<br>
HEADERS – Logs request and response lines and their respective headers. <br>
<br>
NONE – No logs. Use this log level for production environments to enhance the apps performance.

```
val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = HttpLoggingInterceptor.Level.BODY
}
```

<h3> 2. Retry interceptor </h3>
<br>
Retry intercepter is relly useful when API’s are interminent in giving response also when bandwidth is low and API is taking much time to give the response. Below i had a sample interceptor for it. <br>

```
class RetryInterceptor(private val retryAttempts: Int) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        for (i in 1..retryAttempts) {
            try {
                return chain.proceed(chain.request())
            } catch (e: SocketTimeoutException) {
                e.printStackTrace()
            }
        }
        throw RuntimeException("failed to compile the request")
    }
}
```

<h3>3. API Key Interceptor </h3>

API key interceptors are really needed one for most of the realtime projects where you need to add one API Key to make the request so rather than making it many times into many calls have same duplicated code to add a API key we should use an interceptor like below and enjoy it in all requests via OKHttp client :) <br>

```
class APIKeyInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val currentUrl = chain.request().url
        val newUrl = currentUrl.newBuilder().addQueryParameter("api_key", "api_key").build()
        val currentRequest = chain.request().newBuilder()
        val newRequest = currentRequest.url(newUrl).build()
        return chain.proceed(newRequest)
    }
}
```

<h3> 4. Cache interceptor </h3>

If we want to cache the response of the API call so that if we call the API again, the response comes out from Cache. <br>

Let’s say we have the API call from Client to Server and Cache-Control header is enabled from the server, then OkHttp Core will respect that header and cache the response for a certain time being which was sent from the server. <br>

But what if the Cache-Control is not enabled from the server. We still can cache the response from OkHttp Client using Interceptor. <br>
<br>

```
class CacheInterceptor: Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val originalResponse = chain.proceed(chain.request())
        return if (Utils.isNetworkAvailable(applicationContext)) {
            val maxAge = 60
            originalResponse.newBuilder()
                .addHeader("Cache-control", "public, max-age = $maxAge")
                .build()
        } else {
            val maxStale = 60 * 60 * 24 * 28 // 4 weeks
            originalResponse.newBuilder()
                .addHeader("Cache-control", "public, only-if-cache max-age = $maxStale")
                .build()
        }
    }
}
```

<h3> 5. Header via Interceptor </h3>

Let’s say that we have to make the API calls and we have to add Authorization Header in all the API calls. Either we can use it individually or we can centralize that using the Interceptor. <br>
<br>

```
class AuthInterceptor: Interceptor {

    override fun intercept(chain: Interceptor.Chain): Response {
        val currentRequest = chain.request().newBuilder()
        currentRequest.addHeader(AUTHORIZATION, TOKEN)

        val newRequest = currentRequest.build()
        return chain.proceed(newRequest)
    }
}
```
<br>
Implementation of API client in retrofit using all these interceptors. <br>
Here Iam using all the above interceptors in single API client. I know in general we never need all these interceptors in single project but I used all for demo purpose. <br>

```
object RetrofitBuilder {

    private lateinit var retrofit: Retrofit

    fun getRetrofit(): Retrofit {
        if (!this::retrofit.isInitialized) {

            val loggingInterceptor = HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.BODY
            }

            val client = OkHttpClient.Builder()
                .addInterceptor(AuthInterceptor())
                .addInterceptor(loggingInterceptor)
                .addInterceptor(RetryInterceptor(3))
                .addInterceptor(APIKeyInterceptor())
                .build()

            retrofit = Retrofit.Builder()
                .baseUrl(BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .client(client)
                .build()
        }
        return retrofit
    }
}
````

<h1> How viewmodel get retain in configuration changes ?</h1>

Before diving into the internals, let’s revisit what a ‘ViewModel’ is. A ‘ViewModel’ is an architectural component that is used to hold and manage data for views (such as Activities and Fragments) in a way that is conscious of lifecycle events. This helps to separate the responsibilities of data handling and view displaying, leading to more manageable and testable code. <br>

<h3> How ViewModel Works Internally </h3>
When we create a ‘ViewModel’ instance, we usually use ‘ViewModelProvider’, like so:  <br>

```
val viewModel = ViewModelProvider(this).get(MyViewModel::class.java)
```

This ‘ViewModelProvider’ uses a ‘ViewModelStore’ to retain ‘ViewModel’ instances. Let’s have a look at a simplified ‘ViewModelProvider’:  <br>

```
class ViewModelProvider (
  private val store: ViewModelStore, 
  private val factory: Factory
) {
  fun <T: ViewModel> get(modelClass: Class<T>): T {
    var viewModel = store.get(modelClass.name)
  
    if(viewModel == null) {
      viewModel = factory.create(modelClass)
      store.put(modelClass.name, viewModel)
    }
    
    return viewModel as T
  }

  interface Factory {
    fun <T: ViewModel> create(modelClass: Class<T>): T
  }
}
```

In the ‘get’ method, ‘ViewModelProvider’ first tries to get an existing ‘ViewModel’ from the ‘ViewModelStore’ using the class name as the key. If a ‘ViewModel’ with this key doesn’t exist, it creates a new one using the provided ‘Factory’, then stores it in the ‘ViewModelStore’.

The ‘ViewModelStore’ is a simple container that holds ‘ViewModels’. It’s associated with the lifecycle of an ‘Activity’ or ‘Fragment’ and is retained during configuration changes.

```
class ViewModelStore {
  private val map: HashMap<String, ViewModel> = HashMap()
  
  internal fun put(key: String, viewModel: ViewModel) {
    val oldViewModel = map.put(key, viewModel)
    oldViewModel?.onCleared()
  }
  
  internal fun get(key: String): ViewModel? {
    return map[key]
  }
}
```
<br>
When an ‘Activity’ or ‘Fragment’ is created, it will either create a new ‘ViewModelStore’ or retrieve the one that was created during a previous configuration. <br>
<br>
<h3> How ViewModel Survives Configuration Changes </h3> <br>
‘ViewModel’ survives configuration changes due to the way ‘ViewModelStore’ is kept alive by the ‘Activity’ or ‘Fragment’. <br>
<br>
When an ‘Activity’ is recreated (e.g., due to a configuration change like screen rotation), the system keeps the ‘ViewModelStore’ alive and passes it to the new ‘Activity’ instance. This way, when the new ‘Activity’ instance asks for a ‘ViewModel’ of a particular class, the ‘ViewModelProvider’ finds the existing instance in the ‘ViewModelStore’ and returns it, ensuring that the ‘ViewModel’ survives the configuration change. <br>

However, when the ‘Activity’ is finishing (i.e., it’s being destroyed permanently, not due to a configuration change), the ‘ViewModelStore’ is also cleared, and the ‘onCleared’ method is called on all ‘ViewModels’ in the store. <br>

<h1> Difference between aab and apk ?</h1>
<img width="1114" alt="Screenshot 2024-05-10 at 4 02 56 PM" src="https://github.com/siddhpatil6/AndroidBasics/assets/5618021/6cae8f94-98a9-405d-aca1-b92524f7533d">


