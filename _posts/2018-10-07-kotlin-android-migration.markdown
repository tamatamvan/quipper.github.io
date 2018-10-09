---
layout: post
title:  "Kotlin Android migration"
date:   2018-10-07 10:00:00
author: bopbi
comments: true
---

Recently Quipper Android migrated from java to Kotlin, let me share some of the issues that we got during migration process:

## Kotlin gradle configuration/dependencies
For the first step of the migration we followed the [Kotlin Gradle guideline](https://kotlinlang.org/docs/reference/using-gradle.html)

## Existing library problem
The first problem we encountered was a conflict with the [Lombok](https://projectlombok.org/) library during compilation, it always shows error for any property annotated with ```@Getter```. For that reason we need to manually remove the annotation and generate the required getter and setters which consequentially affected a lot of code. The process took almost 2-3 days, and during migration we still maintain the unused getter and setters.

## Existing java source code
[Kotlin visibility](https://kotlinlang.org/docs/reference/visibility-modifiers.html) access is different with java and some *implementation classses* from existing code are designed to be injected, so we have to violate/detour from the existing architecture.

With regards to Java Nullability: the existing code is using a lot of ```Callback``` mechanism and since Java param/property is nullable by default, any property/function converted to Kotlin is nullable unless it proven otherwise.


### Sample Code

Consider this interface that is required to be implemented by a parent Activity of a Fragment:

```java
interface Callback {
  public MyObject getMyObject();
}
```

The Activity and Fragment are coupled by the Callback, but the fragment must always check for nullables.

```java
class MyActivity extends Activity implements Callback {
  private MyObject myObject;

  @Override
  protected void onCreate() {
    if ((this instanceof Callback) == false) {
      throw new Exception("Implement Callback!");
    } 
    // Misc ops
    myObject = new MyObject(miscArgs);
  }

  @Override
  public MyObject getMyObject() {
    return myObject;
  }
}

class MyFragment extends Fragment {
  private Callback callback;

  @Override
  protected void onCreateView(...) {
    if ((getActvity() instanceof Callback) == false) {
      throw new Exception("Implement Callback!");
    }
    callback = (Callback) getActvity();
  }

  @Override
  protected void onResume() {
    // Some task
    MyObject myObject = callback.getMyObject();
    if (myObject != null)
      myObject.someOperation();
  }
}

```

Converted to Kotlin, it carry overs Java's nullable parameters. A Kotlin IDE will then warn of potential NPEs and also suggest some refactors.

```kotlin
class MyActivity : Activity(), Callback {
  private var myObject: MyObject

  override fun onCreate() {
    if (this !is Callback) {
      throw Exception("Implement Callback!")
    }
    // Misc ops
    myObject = MyObject(miscArgs)
  }

  override fun getMyObject(): MyObject {
    return myObject
  }
}

class MyFragment : Fragment() {
  private var callback: Callback

  override fun onCreateView(...) {
    if (getActvity() !is Callback) {
      throw Exception("Implement Callback!")
    }
    callback = getActvity() as Callback
  }

  override fun onResume() {
    // Some task
    val myObject = callback.getMyObject()
    if (myObject != null) {
      myObject.someOperation()
    }
  }
}

```

What Kotlin would look like after more mofications for null safety, notice the `?` operators on objects.

```kotlin
class MyActivity : Activity(), Callback {
  private lateinit var myObject: MyObject

  override fun onCreate() {
    if (this !is Callback) throw Exception("Implement Callback!")

    // Misc ops
    myObject = MyObject(miscArgs)
  }

  override fun getMyObject(): MyObject {
    return myObject
  }
}

class MyFragment : Fragment() {
  private lateinit var callback: Callback

  override fun onCreateView(...) {
    if (getActvity() !is Callback) throw Exception("Implement Callback!")
    callback = getActvity() as Callback
  }

  override fun onResume() {
    // Some task
    callback?.let { cb -> 
      val myObject = cb.getMyObject()
      myObject?.someOperation()
    }
  }
}
```

So until proven otherwise, you must always check for nulls and use safety operators in Kotlin, else refactor the code to explicitly state late initializations and Nullable parameters.
