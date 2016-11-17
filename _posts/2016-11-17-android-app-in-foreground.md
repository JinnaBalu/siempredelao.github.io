---
layout: post
title: "Best way to know if your Android app is currently in foreground"
author: David Aguiar
date: 2016-11-17 18:30:00
categories: Android, UX
highlight: true
comments: true
image: /img/app_foreground_image.jpg
---

Recently, I had to develop a new feature in the app which shows different UI depending whether the app is shown or not. In this post, we will go through different options, from most common, wrong and overused way to the best solution.

Having different behaviors depending if the app is shown or not is a very common use case. For example, let's imagine three case scenarios (although for sure there are many more!):

- <u>Receiving a push notification</u>: refreshing your current displayed UI **Vs** showing a local notification.
- <u>Receiving an incoming call</u>: showing a bubble-Facebook-style to take some notes about the call if app is not shown **Vs** showing the caller data if it is.
- <u>Receiving an email</u>: showing a local notification if the app is not shown **Vs** showing a Snackbar if it is.
- ...

### First approach

Our first approach will use the **getRunningTasks** method from the **ActivityManager** class. The **getRunningTasks** method returns a list of the tasks that are currently running, with the most recent being first and older ones after in order. Sooo... perfect! That's exactly what we are looking for, we just want to check if the most recent task coincides with ours. Moreover, this method is available since API 1 (Android 1.0 - Alpha), so it will be backwards compatible to the beginning of the world! All hands on deck! 

```java
public static boolean isApplicationInForeground(final Context context) {
    final ActivityManager activityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    final List<RunningTaskInfo> tasks = activityManager.getRunningTasks(1);
    if (!tasks.isEmpty()) {
        final ComponentName topActivity = tasks.get(0).topActivity;
        if (topActivity.getPackageName().equals(context.getPackageName())) {
            return true;
        }
    }
    return false;
}
```

This method needs the **GET_TASKS** permission in order to work, so we add it to our *AndroidManifest* file:

```xml
<manifest package="your.package.name"
          xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.GET_TASKS"/>

    <!-- ... -->
</manifest>
```

First, we get the ActivityManager from the SystemService and then, with the **getRunningTasks** method parameter **1**, we get only one running task, that is, the most recent. Finally, we check if the Activity shown package name of the most recent task coincides with our app package name, in other words, that Activity belongs to our app.

#### All that is gold does not glitter

I see a problem here: could you imagine what would happen in a multi-window environment like, by default, Android N or any other previous manufacturer's fork implementation (for example, Sony already has multi-window in Marshmallow under certain circumstances)? If we have two apps shown, and our app is the second in this list, after running this method we will get false, but it is indeed shown!

Also, this method got deprecated in API 21 (Android 5.0 - Lollipop), not even the **GET_TASKS** permission is needed. It will still work in pre-Lollipop devices, but will return a small subset of its data: at least the caller's own tasks and possibly some other tasks such as home that are known to not be sensitive.

There is an important note in the documentation:

> Note: this method is only intended for debugging and presenting task management user interfaces. This should never be used for core logic in an application, such as deciding between different behaviors based on the information found here. Such uses are not supported, and will likely break in the future. For example, if multiple applications can be actively running at the same time, assumptions made about the meaning of the data here for purposes of control flow will be incorrect.

### Second approach

The second approach will use the **getRunningAppProcesses** method also from the **ActivityManager** class. The **getRunningAppProcesses** returns a list of application processes that are running on the device, in this case not in MRU (most recently used) or any specific order. However, this method is available since API 3 (Android 1.5 - Cupcake), although I think nowadays it doesn't make a big difference from API 1 to API 3 because a 0% of devices runs in both API levels (regarding [Android dashboard distribution on November 2016][november_dashboard]). Let's see the implementation:

```java
public static boolean isApplicationInForeground(final Context context) {
    final ActivityManager activityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    final List<ActivityManager.RunningAppProcessInfo> runningProcesses = activityManager.getRunningAppProcesses();
    for (final ActivityManager.RunningAppProcessInfo processInfo : runningProcesses) {
        if (processInfo.importance == ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND) {
            for (final String activeProcess : processInfo.pkgList) {
                if (activeProcess.equals(context.getPackageName())) {
                    return true;
                }
            }
        }
    }
    return false;
}
```

Here we have followed a similar process like in first approach: first, we get the list of application processes; for each process, we check if it is in foreground and finally if the package name coincides with our app package name.

#### All that is gold does not glitter

Here we have also the same problem than in previous approach, this method is only intended to show the user the running apps, being useful, for example, in a task manager app. Besides, calling this method from the application UI thread will return IMPORTANCE_FOREGROUND even if it is not in foreground; doing that from an application background thread will return correct results.

Like in the first approach, there is an important note in the documentation:

> Note: this method is only intended for debugging or building a user-facing process management UI.

### Final approach

The final approach will use the interface **ActivityLifecycleCallbacks** available in the **Application** class. This interface contains several method callbacks that are executed in each particular state of the Activity. Have a look:

```java
public class Application extends ContextWrapper implements ComponentCallbacks2 {
    // ...
    public interface ActivityLifecycleCallbacks {
        void onActivityCreated(Activity activity, Bundle savedInstanceState);
        void onActivityStarted(Activity activity);
        void onActivityResumed(Activity activity);
        void onActivityPaused(Activity activity);
        void onActivityStopped(Activity activity);
        void onActivitySaveInstanceState(Activity activity, Bundle outState);
        void onActivityDestroyed(Activity activity);
    }
}
```

In our particular case, we are interested whether or not an Activity is shown. To do this, the interesting Activity lifecycle methods are **onResume** and **onStop**. Per the documentation, Resumed (after executing onResume) and Stopped (after executing onStop) states are defined as follows:

> Resumed: In this state, the activity is in the foreground and the user can interact with it. (Also sometimes referred to as the "running" state.)
>
> Stopped: In this state, the activity is completely hidden and not visible to the user; it is considered to be in the background. While stopped, the activity instance and all its state information such as member variables is retained, but it cannot execute any code.

So let's implement only the ActivityLifecycleCallbacks **onActivityResumed** and **onActivityStopped** methods (the other methods will be empty and, to make this example simpler, will not be shown here):

```java
public class MyActivityLifecycleCallback implements Application.ActivityLifecycleCallbacks {

    private static int foregroundActivityCount = 0;

    // ...

    @Override
    public void onActivityResumed(final Activity activity) {
        foregroundActivityCount++;
    }

    @Override
    public void onActivityStopped(final Activity activity) {
        foregroundActivityCount--;
    }

    public static boolean isApplicationInForeground() {
        return foregroundActivityCount > 0;
    }
}
```

##### But wait... why don't you use a boolean value instead of this integer counter? Why is this a better implementation than using a boolean?

When an *Activity A* opens another *Activity B*, the first executed method is *ActivityB.onResume* and then *ActivityA.onStop*. If we use a boolean value, let's say with this implementation:

```java
    private static boolean isShown = false;

    @Override
    public void onActivityResumed(final Activity activity) {
        isShown = true;
    }

    @Override
    public void onActivityStopped(final Activity activity) {
        isShown = false;
    }
}
```

, at the end of this execution we will get a false value from **isApplicationInForeground** because *ActivityA.onStop* will be the last method to be executed and will set this to false. In other words:

1. ActivityA.onResume -> isShown == true
2. ActivityB.onResume -> isShown == true
3. ActivityA.onStop -> isShown == false
4. isApplicationInForeground() == false :(

, whereas with our integer counter implementation, we will have this result:

1. ActivityA.onResume -> foregroundActivityCount == 1
2. ActivityB.onResume -> foregroundActivityCount == 2
3. ActivityA.onStop -> foregroundActivityCount == 1
4. isApplicationInForeground() == true :)

<br>

Once made this clarification, next step is registering this lifecycle callback in our Application class. To do so, first you have to extend your Application class and then use the **registerActivityLifecycleCallbacks** method available in the Application class.

```java
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();

        // ...

        registerActivityLifecycleCallbacks(new MyActivityLifecycleCallback());
    }
}
```

Remember to add the *name* attribute to your *application* tag in the *AndroidManifest* file:

```xml
<manifest package="your.package.name"
          xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- ... -->
    <application android:name=".MyApplication">
        <!-- ... -->
    </application>

</manifest>
```

Then, from a service or background thread, we can ask if the app is currently in foreground using the **isApplicationInForeground** method.

```java
public class MyServiceOrMyBackgroundThread {

    public void foo() {
        if (MyActivityLifecycleCallback.isApplicationInForeground()) {
            // start a new Activity? show a Snackbar? show a Toast?
        } else {
            // create a notification? create a bubble Facebook-style?
        }
    }
}
```

#### All that is gold does not glitter

Yes, even the final approach has some caveats. Although the **GET_TASKS** permission is not needed anymore, this last approach will only be available, and then, will only work, since API 14 (Android 4.0 - Ice Cream Sandwich). The good side is that targeting *minSdkVersion 14* will point to the 98,6% of devices worldwide as of November 2016 (check [Android dashboard distribution on November 2016][november_dashboard]).

<br><br><br>

##### Want to know more?

- [ActivityManager::getRunningTasks] [get_running_tasks]
- [ActivityManager::getRunningAppProcesses] [get_running_app_processes]
- [Application.ActivityLifecycleCallbacks] [activity_lifecycle_callbacks]
- [Activity Lifecycle][activity_lifecycle]




[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job)

[get_running_tasks]: https://developer.android.com/reference/android/app/ActivityManager.html#getRunningTasks(int)
[get_running_app_processes]: https://developer.android.com/reference/android/app/ActivityManager.html#getRunningAppProcesses()
[activity_lifecycle_callbacks]: https://developer.android.com/reference/android/app/Application.ActivityLifecycleCallbacks.html
[activity_lifecycle]: https://developer.android.com/training/basics/activity-lifecycle/starting.html
[november_dashboard]: http://www.droid-life.com/2016/11/07/android-distribution-updated-november-2016-nougat-debuts/ "Android Distribution 11.2016"
