---
layout: post
title: "Decorator pattern and an Android use case"
author: David Aguiar
date: 2017-03-14 08:00:00
categories: Java, Desing patterns, Decorator, Android
highlight: true
comments: true
image: /img/art-wall-brush-painting.jpg
---

Decorator pattern is a structural design pattern that allows us to __dinamically__ add responsibilities to concrete objects, by using _composition_: that is to say, this pattern _decorates_ objects to give them more behaviour than what they have when they are created and each decorator adds its own behaviour and responsibilities, fitting this in the Single Responsibility Principle. This pattern offers a more flexible alternative to extend behaviour than _inheritance_.

### What happens with Inheritance?

Let's imagine we want to model all kind of drinks we have available in a bar. As each drink has specific ingredients, we would start modelling each drink in a separated class. For example:

```java
interface Drink {
    String serve();
}

class DaiquiriDrink extends Drink {
    public String serve() {
        return "Please, a drink with:\n- white rum\n- lime juice\n- sugar\n- ice";
    }
}
```

Ok, so far so good. We could think that it's not that bad to have, let's say, 30 classes like DaiquiriDrink class, one per each kind of drink (ManhattanDrink, MargaritaDrink, CosmopolitanDrink, ...). But now imagine that a customer comes to the bar and orders a daiquiri without sugar. Using inheritance, we would have to create a new class DaiquiriWithoutSugarDrink extending the previous one:

```java
class DaiquiriWithoutSugarDrink extends DaiquiriDrink {
    public String serve() {
        return "Please, a drink with:\n- white rum\n- lime juice\n- ice";
    }
}
```

Or another customer comes and orders a double rum daiquiri:

```java
class DaiquiriDoubleRumDrink extends DaiquiriDrink {
    public String serve() {
        return "Please, a drink with:\n- double white rum\n- lime juice\n- sugar\n- ice";
    }
}
```

Or another customer comes and orders a double rum without sugar daiquiri:

```java
class DaiquiriDoubleRumWithoutSugarDrink extends DaiquiriDoubleRumDrink {
    public String serve() {
        return "Please, a drink with:\n- double white rum\n- lime juice\n- ice";
    }
}
```

I think you can now perceive the explotion of combinations of this approach. If we want to model our business model, we would have to create a big amount of inherited classes, one per each drink-ingredients-combination, leading to a complex hierarchy. 

### How is this done with Decorator pattern?

With decorators, you only model the ingredient once and then you can freely combine it with a much cleaner result. To do that, we need a __base class__ (_BaseDrink_ in this example) that is going to act as the main component from where to extend behaviour; a __decorator abstraction__ (_DrinkDecorator_ in this example) and whichever amount of __decorator implementations__ we need to extend behaviour. Have a look at the code:

```java
// Base class from where to start
class BaseDrink implemens Drink {
    public String serve() {
        return "Please, a drink with:";
    }
}

// abstract, we don't want to create decorator abstraction objects but subclasses
abstract class DrinkDecorator implements Drink {

    private final Drink decoratedDrink;

    public DrinkDecorator(Drink drink) {
        this.decoratedDrink = drink;
    }

    public String serve() {
        return decoratedDrink.serve();
    }
}
```

Now, we are ready to create specific drink decorators!

```java
class IceDecorator extends DrinkDecorator {

    public IceDecorator(Drink drink) {
        super(drink);
    }

    public String serve() {
        return super.serve() + "\n- ice";
    }
}

class SugarDecorator extends DrinkDecorator {

    public SugarDecorator(Drink drink) {
        super(drink);
    }

    public String serve() {
        return super.serve() + "\n- sugar";
    }
}

// ... as many decorators as you need
```

And finally, here is how we would use the decorators:

```java
public class Main {
    public static void main(String[] args) {
        Drink daiquiriDrink = new IceDecorator(new SugarDecorator(new LimeJuiceDecorator(new WhiteRumDecorator(new BaseDrink()))));
        Drink daiquiriWithoutSugarDrink = new IceDecorator(new LimeJuiceDecorator(new WhiteRumDecorator(new BaseDrink())));
        Drink daiquiriDoubleRumDrink = new IceDecorator(new SugarDecorator(new LimeJuiceDecorator(new WhiteRumDecorator(new WhiteRumDecorator(new BaseDrink())))));
        Drink daiquiriDoubleRumWithoutSugarDrink = new IceDecorator(new LimeJuiceDecorator(new WhiteRumDecorator(new WhiteRumDecorator(new BaseDrink()))));

        daiquiriDrink.serve();
        daiquiriWithoutSugarDrink.serve();
        daiquiriDoubleRumDrink.serve();
        daiquiriDoubleRumWithoutSugarDrink.serve();
    }
}
```

, printing in console the following message:

```
Please, a drink with:
- white rum
- lime juice
- sugar
- ice
Please, a drink with:
- white rum
- lime juice
- ice
Please, a drink with:
- white rum
- white rum
- lime juice
- sugar
- ice
Please, a drink with:
- white rum
- white rum
- lime juice
- ice
```

As we have seen, decorator pattern allows us not only to add but also to withdraw behaviour without writing a lot of code. Although we could have achieved the same result by using inheritance, we wouldn't have the same flexibility than with this solution.

But not all are pros: using decorators leads to small classes but a large amount of these ones. And chaining several decorators can make debugging hard.

### Use case in Android

Few months ago, to provide better user feedback in my [pet project], I thought it was a good idea to also gather some device data. In Android, we can gather device information using the [Build] class and some other mechanisms and, since API 16, it is also possible to gather memory information through the [ActivityManager.MemoryInfo] class. So I thought it was a good fit for the decorator pattern: we want to add ("decorate") memory information to already available device information. Let's see how.

First, we create the main component and base implementation.

```java
public interface DeviceInfo {
    String getDeviceInfo();
}

public class DeviceInfoBase implements DeviceInfo {

    private final Context        context;
    private final PackageManager packageManager;

    // PackageManager is just a wrapper over Android's PackageManager for unit tests purposes 
    public DeviceInfoBase(final Context context, final PackageManager packageManager) {
        this.context = context;
        this.packageManager = packageManager;
    }

    @Override
    public String getDeviceInfo() {
        return String.format(Locale.getDefault(),
                             "Important device info for analysis:\n\nVersion:\nNAME=%s\nRELEASE=%s\nSDK_INT=%d\n\nDevice:\nMANUFACTURER=%s\nBRAND=%s\nMODEL=%s\nDEVICE=%s\nPRODUCT=%s\nDENSITY_DPI=%d\n\nOther:\nBOARD=%s\nBOOTLOADER=%s\nDISPLAY=%s\nFINGERPRINT=%s\nHARDWARE=%s\nHOST=%s\nID=%s\nTAGS=%s\nTIME=%d\nTYPE=%s\nUSER=%s",
                             packageManager.getVersionName(),
                             Build.VERSION.RELEASE,
                             Build.VERSION.SDK_INT,
                             Build.MANUFACTURER,
                             Build.BRAND,
                             Build.MODEL,
                             Build.DEVICE,
                             Build.PRODUCT,
                             context.getResources().getDisplayMetrics().densityDpi,
                             Build.BOARD,
                             Build.BOOTLOADER,
                             Build.DISPLAY,
                             Build.FINGERPRINT,
                             Build.HARDWARE,
                             Build.HOST,
                             Build.ID,
                             Build.TAGS,
                             Build.TIME,
                             Build.TYPE,
                             Build.USER);
    }
}
```

Now, we create the device info decorator abstraction and the API 16 decorator implementation.

```java
public abstract class DeviceInfoDecorator implements DeviceInfo {

    private final DeviceInfo decoratedDeviceInfo;

    public DeviceInfoDecorator(final DeviceInfo decoratedDeviceInfo) {
        this.decoratedDeviceInfo = decoratedDeviceInfo;
    }

    @Override
    public String getDeviceInfo() {
        return decoratedDeviceInfo.getDeviceInfo();
    }
}

@RequiresApi(api = JELLY_BEAN)
public class DeviceInfoApi16Decorator extends DeviceInfoDecorator {

    private final MemoryInfo memoryInfo;

    // MemoryInfo is a wrapper over Android's ActivityManager.MemoryInfo for unit tests purposes 
    public DeviceInfoApi16Decorator(final DeviceInfo decoratedDeviceInfo, final MemoryInfo memoryInfo) {
        super(decoratedDeviceInfo);
        this.memoryInfo = memoryInfo;
    }

    @Override
    public String getDeviceInfo() {
        return String.format(Locale.getDefault(), "%s\n%s", super.getDeviceInfo(), getMemoryParameters());
    }

    private String getMemoryParameters() {
        final long freeMemoryMBs = memoryInfo.getFreeMemory();
        final long totalMemoryMBs = memoryInfo.getAvailableMemory();

        return String.format(Locale.getDefault(),
                                 "TOTALMEMORYSIZE=%dMB\nFREEMEMORYSIZE=%dMB",
                                 totalMemoryMbs,
                                 freeMemoryMbs);
    }
}
```

Et voilà! Finally, in our DI module:

```java
@Provides
@Singleton
DeviceInfo getDeviceInfo(Context context, PackageManager packageManager, MemoryInfo memoryInfo) {
    final DeviceInfoBase deviceInfoBase = new DeviceInfoBase(context, packageManager);
    if (SDK_INT < JELLY_BEAN) {
        return deviceInfoBase;
    } else {
        return new DeviceInfoApi16Decorator(deviceInfoBase, memoryInfo);
    }
}
```

You can see the real example in my [GitHub repo].

<br><br><br>

##### Want to know more?

- [Decorator pattern (Wikipedia)] [wikipedia]
- [Decorator pattern (JournalDev)] [journaldev]




[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job)

[wikipedia]: https://en.wikipedia.org/wiki/Decorator_pattern "Open Decorator pattern in Wikipedia"
[journaldev]: http://www.journaldev.com/1540/decorator-design-pattern-in-java-example "Open Decorator pattern in JournalDev"
[pet project]: https://play.google.com/store/apps/details?id=gc.david.dfm "Distance From Me at Google Play Store"
[ActivityManager.MemoryInfo]: https://developer.android.com/reference/android/app/ActivityManager.MemoryInfo.html
[Build]: https://developer.android.com/reference/android/os/Build.html
[GitHub repo]: https://github.com/siempredelao/Distance-From-Me-Android/tree/master/app/src/main/java/gc/david/dfm/deviceinfo
