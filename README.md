# AndroidBasics
Android Related Questions
<h3>SSL Pinning in Android </h3>
<h3>Broadcast Receiver </h3>
<h3> Retrofit Interceptors for Beginners </h3>
<h3> How viewmodel get retain in configuration changes ? </h3>
<h3> Difference between aab and apk ?</h3>
<h3>why stackoverflow exception occurs </h3>
<h3>Explain Launch Modes </h3>

<h2>SSL Pinning in Android (Simplified Explanation)</h2>

<h3>What is SSL?</h3>
<p>SSL (Secure Sockets Layer) ensures that data sent between your Android app and a server is <strong>encrypted</strong> and <strong>secure</strong>. It prevents hackers from spying on sensitive information like passwords or payment details.</p>

<h3>What is SSL Pinning?</h3>
<p>SSL Pinning is an extra security layer where the app <strong>remembers</strong> (or "pins") a specific SSL certificate. Instead of trusting any certificate the server presents, the app <strong>only trusts</strong> the pinned certificate.</p>

<h3>Why is SSL Pinning Needed?</h3>
<p>Without SSL Pinning, a hacker can use a <strong>fake certificate</strong> to intercept (MITM attack) your communication and steal sensitive data.</p>

<h3>How SSL Pinning Works?</h3>
<ol>
    <li>The app stores the server’s SSL certificate (or public key).</li>
    <li>When connecting to the server, the app <strong>compares</strong> the server’s certificate with the stored one.</li>
    <li>If they match ✅ → Connection continues.</li>
    <li>If they don’t ❌ → Connection is rejected (prevents fake certificates).</li>
</ol>

<h3>How to Implement SSL Pinning in Android?</h3>

<h4>Certificate Pinning (Recommended) using OkHttp</h4>
<pre><code>val client = OkHttpClient.Builder()
    .certificatePinner(
        CertificatePinner.Builder()
            .add("yourserver.com", "sha256/your_certificate_hash")
            .build()
    )
    .build()</code></pre>
<p>- Replace <code>yourserver.com</code> with your server's domain.</p>
<p>- Replace <code>"sha256/your_certificate_hash"</code> with your actual certificate fingerprint.</p>

<h4>Public Key Pinning (Alternative)</h4>
<p>Instead of pinning the entire certificate, you can pin just the <strong>public key</strong> inside it.</p>

<h3>Pros of SSL Pinning</h3>
<ul>
    <li>✅ Protects against Man-in-the-Middle (MITM) attacks.</li>
    <li>✅ Ensures the app only communicates with the real server.</li>
</ul>

<h3>Cons of SSL Pinning</h3>
<ul>
    <li>❌ If the certificate changes, the app must be updated.</li>
    <li>❌ Misconfigured pinning can break the app’s network communication.</li>
</ul>

<h3>When to Use SSL Pinning?</h3>
<ul>
    <li>Banking apps</li>
    <li>Payment apps</li>
    <li>Apps handling sensitive data (health, finance, etc.)</li>
</ul>

<p>🚀 <strong>In simple words, SSL Pinning ensures your app talks only to the real server, preventing hackers from listening in.</strong></p>

<h2>Best Ways to Avoid Memory Leaks in Android (Simplified)</h2>

<h3>1. Use applicationContext Instead of activityContext</h3>
<p>✅ <strong>Why?</strong> Activity context holds a reference to the activity, which can cause memory leaks if the activity is destroyed.</p>
<pre>
// ❌ Bad Practice (Leads to memory leaks)
val myToast = Toast.makeText(this, "Hello", Toast.LENGTH_SHORT)

// ✅ Good Practice (Uses application context)
val myToast = Toast.makeText(applicationContext, "Hello", Toast.LENGTH_SHORT)
</pre>

<h3>2. Use Weak References for Long-Lived Objects</h3>
<p>✅ <strong>Why?</strong> If an object lives longer than an activity (like a background task), it can cause a leak. Use <code>WeakReference</code>.</p>
<pre>
class MyTask(context: Context) {
    private val weakContext = WeakReference(context)

    fun doSomething() {
        val context = weakContext.get()
        context?.let {
            Toast.makeText(it, "Task Done", Toast.LENGTH_SHORT).show()
        }
    }
}
</pre>

<h3>3. Avoid Storing Large Objects in Static Variables</h3>
<p>✅ <strong>Why?</strong> Static variables live as long as the app, so they prevent objects from being garbage-collected.</p>
<pre>
// ❌ Bad Practice
companion object {
    var bitmap: Bitmap? = null  // Can cause a memory leak
}

// ✅ Good Practice
companion object {
    private var weakBitmap: WeakReference<Bitmap>? = null
}
</pre>

<h3>4. Use ViewBinding Instead of findViewById</h3>
<p>✅ <strong>Why?</strong> <code>findViewById</code> holds references longer than necessary, causing leaks. <code>ViewBinding</code> helps avoid this.</p>
<pre>
class MyActivity : AppCompatActivity() {
    private var _binding: ActivityMainBinding? = null
    private val binding get() = _binding!!

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        _binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }

    override fun onDestroy() {
        super.onDestroy()
        _binding = null  // Prevent memory leak
    }
}
</pre>

<h3>5. Unregister Listeners & Observers in onDestroy()</h3>
<p>✅ <strong>Why?</strong> Registered listeners keep references to activities, preventing them from being garbage collected.</p>
<pre>
override fun onDestroy() {
    super.onDestroy()
    myButton.setOnClickListener(null)  // Avoid memory leak
}
</pre>

<h3>6. Use Lifecycle-Aware Components</h3>
<p>✅ <strong>Why?</strong> Lifecycle-aware components automatically clean up resources when the activity is destroyed.</p>
<pre>
class MyViewModel : ViewModel() {
    val data = MutableLiveData<String>()

    override fun onCleared() {
        super.onCleared()
        // Clean up resources here
    }
}
</pre>

<h3>7. Use LeakCanary for Debugging</h3>
<p>✅ <strong>Why?</strong> LeakCanary helps detect memory leaks in your app.</p>
<pre>
dependencies {
    debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.10'
}
</pre>

<h3>Conclusion</h3>
<ul>
  <li>✅ Use <code>applicationContext</code> where possible</li>
  <li>✅ Avoid static references to large objects</li>
  <li>✅ Use <code>WeakReference</code> for long-lived objects</li>
  <li>✅ Unregister listeners in <code>onDestroy()</code></li>
  <li>✅ Use <code>ViewBinding</code> instead of <code>findViewById()</code></li>
  <li>✅ Use lifecycle-aware components</li>
  <li>✅ Use LeakCanary to detect leaks</li>
</ul>
<p>By following these practices, you can <strong>prevent memory leaks and keep your app efficient</strong>! 🚀</p>


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
NONE – No logs. Use this log level for production environments to enhance the apps performance. <br>

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

<h1> How viewmodel get retain in configuration changes ? </h1>

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

In the ‘get’ method, ‘ViewModelProvider’ first tries to get an existing ‘ViewModel’ from the ‘ViewModelStore’ using the class name as the key. If a ‘ViewModel’ with this key doesn’t exist, it creates a new one using the provided ‘Factory’, then stores it in the ‘ViewModelStore’.  <br>

The ‘ViewModelStore’ is a simple container that holds ‘ViewModels’. It’s associated with the lifecycle of an ‘Activity’ or ‘Fragment’ and is retained during configuration changes.  <br>

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


<h1>why stackoverflow exception occurs </h1>
A StackOverflowError occurs when a program recurses too deeply, causing the stack to overflow. This typically happens in situations where a function calls itself recursively without a proper termination condition, or when the recursion depth is too large for the available stack space. <br>

Here's a general overview of how a StackOverflowError occurs: <br>

Function Calls: When a function is called, a stack frame is created to store the function's local variables and execution context. This stack frame is added to the call stack. <br>
 <br>
Recursive Calls: In recursive functions, a function calls itself. Each recursive call adds a new stack frame to the call stack. <br>
 <br>
Limited Stack Space: The call stack has a limited amount of space. If a program recurses too deeply, adding more stack frames than the available space, it causes a stack overflow. <br>
 <br>
StackOverflowError: When the stack overflows, the program throws a StackOverflowError, indicating that the call stack has exceeded its capacity. <br>
 <br>
To prevent StackOverflowError, ensure that your recursive functions have a proper termination condition to stop the recursion. Also, consider using iterative solutions or optimizing your recursive algorithms to reduce the depth of recursion. <br>


<h1>Explain Launch Modes : </h1>

<h2> 1. standard Launch Mode </h2>
Default mode for activities. <br>
Each time the activity is started, a new instance is created and pushed onto the task's stack. <br>
<br>
Example: <br>
<br>
Start A → A is created. <br>
Start B from A → B is created and placed on top of A. <br>
Start C from B → C is created and placed on top of B. <br>
Start A from C → Another instance of A is created and placed on top of C. <br>

<h3> Stack: </h3>

```
A → B → C → A (new)
```

<h2> 2. singleTop Launch Mode </h2>
<br>
If an instance of the activity is already at the top of the stack, a new instance is not created. Instead, the existing instance handles the intent via onNewIntent().<br>
<br>
Example:<br>
<br>
Start A → A is created. <br>
Start B from A → B is created and placed on top of A. <br>
Start C from B → C is created and placed on top of B. <br>
Start C from C → Instead of creating a new instance, the existing C handles the intent. <br>

<h3>Stack: </h3>

```
A → B → C
```

<h2> 3. singleTask Launch Mode </h2> 
If an instance of the activity exists in the task, it is brought to the front, and all activities above it are cleared.
The existing instance handles the new intent via onNewIntent(). <br>
<br>
Example:<br>
<br>
Start A → A is created.<br>
Start B from A → B is created and placed on top of A.<br>
Start C from B → C is created and placed on top of B.<br>
Start A from C → The existing instance of A is brought to the front, and B and C are removed from the stack.<br>
<br>

<h3>Stack:</h3>

```
A
```

<h2>4. singleInstance Launch Mode </h2>
<br>
Similar to singleTask, but the activity runs in its own separate task.<br>
No other activities can be started in the same task.<br>
<br>
Example:<br>
Start A → A is created.<br>
Start B from A → B is created and placed on top of A.<br>
Start C from B → C is created and placed on top of B.<br>
Start D (with singleInstance mode) from C → D is created in a new task.<br>
Task Stack:<br>

```
Task 1: A → B → C
Task 2: D
```
