# API and JSON

API stands for Application Programming Interface. An API is a URL. if you call it, it is going to give the result in JSON format. 

Whenever someone wants to share data, they can provide an API. If you hit that API, it can give you the data in JSON format.

# JSON

JSON stands for Java Script Object Notion (JSON).

It is a data exchanging format between your Webservice and your Android app.

#### **JSON Syntax:**

```json
{“Key”: “Value”}
```

Key will always be a string

Value can be a string, a number, an object, an Array! It can be anything. It is possible to have an object that contains multiple objects and arrays.

- We refer to Objects as JsonObject -> denoted with Curly braces {}

- Arrays are called JsonArrays -> denoted with Curly square brackets []

##### **Example JSON format data:**

```json
{“Name”:”CodingDojo”} simple JSON example with value as single string
{ “Age” : 26 } simple JSON example with value as single number
{“Details” : {“Name”:”CodingDojo”} } simple JSON example with value as a single JsonObject
{“Users”: [
       {“Name”:”CodingDojo”} ,
       {“Name”:”CodingDojo”} ,
       {“Name”:”CodingDojo”} 
       ]
} simple JSON example with value as JsonArray
```

Notice that a single JsonArray can have multiple objects inside

## Retrofit

In order to work with Retrofit, let’s divide the steps in using Retrofit HttpClient.

- Add the dependencies in the dependency section 
- Understand your result and based on that create a Model class, to hold data
- Declare and initialize the Retrofit instance 
- Start getting data from Retrofit!
- Use the data in your Android app.

##### **Adding Dependencies:**

In the build.gradle file (Module level) you will have to add the following decencies. 

```kotlin
implementation 'com.google.code.gson:gson:2.8.8'
implementation 'com.squareup.initializeBinding:retrofit:2.9.0' 
implementation 'com.squareup.retrofit2:converter-gson:2.9.0' 
implementation 'com.squareup.okhttp3:logging-interceptor:4.2.2'
```

##### Manifest

Add internet permission in the AndroidManifest file above the <application> tag

```xml
<uses-permission android:name="android.permission.INTERNET"></uses-permission>
```

#####  Interface

```kotlin
import retrofit2.Call
import retrofit2.http.GET

interface APIInterface {
    @GET("advice")
    fun getAdvice(): Call<Advice>
}
```

##### Retrofit client class

you can use object instead of class

```kotlin
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

class APIClient {
    private var retrofit : Retrofit? = null
    fun getClient() : Retrofit? {
        retrofit = Retrofit.Builder()
            .baseUrl("https://api.adviceslip.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
        return retrofit
    }
}
```

## Coroutines

For more on threads, [visit the official documentation](https://developer.android.com/reference/java/lang/Thread).

A Coroutine runs jobs in the background thread in an efficient way that will solve the time-consuming operations problem in the main thread.

Go to this link and find the latest [Dependency](https://developer.android.com/kotlin/coroutines)

Copy the Dependency and paste it in the “build.gradle”:

```
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9")
}
```

To run time-consuming code in the background using Coroutines we need to define the context “Dispatchers” of the background work which can be (IO, Main, Default).

After defining the context, we can start our coroutines by creating its builder (Launch):

```
CoroutineScope(IO).launch {}
```

We also have globalScope which will keep running as long as your application runs, And lifecycleScope which is bound to the Activity or Fragment lifecycle.

The time-consuming code will be written inside the two curly brackets.

Next, we will create a function which will be used and controlled by coroutines:

```
private suspend fun getResult(){}
```

The suspend modifier is added to the function to define that it can be stopped/resumed by the Coroutines and it can only get called inside the Coroutines scope.

If we need to access views from within Coroutines then we need to change the context of the job to be in the main thread by using withContext(Main).

```
withContext(Main){}
```

### retrofit with coroutines

do all the steps for retrofit and coroutines: dependencies, manifest, interface and retrofit client. 

#### Main

```kotlin

    private fun requestAPI(){
        CoroutineScope(IO).launch {
            Log.d("MAIN", "fetch advice")
            val advice = async { fetchAdvice() }.await()
            if(advice.isNotEmpty()){ updateTextView(advice) }
            else{ Log.d("MAIN", "Unable to get data") }
        }
    }


    private fun fetchAdvice(): String{
        Log.d("MAIN", "went inside fetch")
        val apiInterface = APIClient().getClient()?.create(APIInterface::class.java)
        val call: Call<Advice> = apiInterface!!.getAdvice()
        var advice = ""

        try {
            val response = call.execute()
            advice = response.body()?.slip?.advice.toString()
            Log.d("MAIN", "read advice")
        }
        catch (e: Exception){Log.d("MAIN", "ISSUE: $e")}


        Log.d("MAIN", "advice is $advice")
        return advice
    }

    private suspend fun updateTextView(advice : String) {
        withContext(Main){
            tvAdvice.text = advice
        }
    }
```

