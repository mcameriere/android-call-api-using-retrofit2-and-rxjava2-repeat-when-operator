# android-call-api-using-retrofit2-and-rxjava2-repeat-when-operator

* Open **Android Studio** and click **Start a new Android Studio project**
* In **Select a Project Template** choose **Empty Activity** and click **Next**
* Name your project
* Choose language **Kotlin**
* Click **Finish**

## app/build.gradle

    dependencies {
        ...
        implementation 'io.reactivex.rxjava2:rxjava:2.2.10'
        implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
        implementation 'com.squareup.retrofit2:retrofit:2.5.0'
        implementation 'com.squareup.retrofit2:converter-gson:2.5.0'
        implementation 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
        implementation 'com.squareup.okhttp3:logging-interceptor:3.12.0'
    }

## AndroidManifest.xml

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## SequenceNumberResponse.kt

```java
class SequenceNumberResponse(val value: Int)
```

## IntenseTundraService.kt

```java
import io.reactivex.Observable
import retrofit2.http.GET

interface IntenseTundraService {
    @GET("sequence")
    fun getSequenceNumber(): Observable<SequenceNumberResponse>
}
```

## activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/buttonStart"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Start"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## MainActivity.kt

```java
val httpLoggingInterceptor = HttpLoggingInterceptor().apply {
    level = HttpLoggingInterceptor.Level.BODY
}

val okHttpClient = OkHttpClient.Builder()
        .addInterceptor(httpLoggingInterceptor)
        .build()

val retrofit = Retrofit.Builder()
        .baseUrl("https://your-url-here.com")
        .client(okHttpClient)
        .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
        .addConverterFactory(GsonConverterFactory.create())
        .build()

val service = retrofit.create(IntenseTundraService::class.java)

val twoSecondsObservable = service.getSequenceNumber()
    .repeatWhen { completed -> completed.delay(2, TimeUnit.SECONDS) }
    .takeUntil { it.value == 4 }

val fiveSecondsObservable = service.getSequenceNumber()
    .repeatWhen { completed -> completed.delay(5, TimeUnit.SECONDS) }
    .takeUntil { it.value == 9 }

buttonStart.setOnClickListener {

    Observable
        .concat(
            twoSecondsObservable,
            fiveSecondsObservable
        )
        .subscribeOn(Schedulers.io())
        .subscribe { Log.d("TAG", "value: ${it.value}") }

}
```
