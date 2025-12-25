In Jetpack Compose, managing state is about deciding how long a piece of information should "live."

### remember vs rememberSaveable

| Feature        | `remember`                                     | `rememberSaveable`                                                |
|----------------|------------------------------------------------|-------------------------------------------------------------------|
| **Survival**   | Survives Recomposition.                        | Survives Recomposition **+** Screen Rotation **+** Process Death. |
| **Storage**    | In-memory (RAM).                               | Bundle (saved on disk/system memory).                             |
| **Data Types** | Anything (Objects, Lambdas, etc.).             | Only "Saveable" types (Primitives, Parcelable, Serializable).     |
| **Best For**   | Transient UI state (e.g., is a dropdown open). | User-entered data (e.g., text in a form).                         |

---

### How They Work Internally (Easy Version)

#### 1. `remember`: The Slot Table

Think of `remember` as a **sticky note** on a specific part of your UI tree.

* **The Mechanism:** Compose maintains a "Slot Table" (a flat data structure that stores the state of your UI).
* **The Process:** When a Composable runs for the first time, `remember` calculates the value and stores it in a "slot."
  During the next recomposition, Compose looks at that same slot and says, "Oh, I already have a value for this," and
  skips the calculation.
* **The Limit:** If the Activity is destroyed (like when you rotate the screen), the entire Slot Table is cleared. Your
  sticky note is gone.

#### 2. `rememberSaveable`: The Registry & Bundle

Think of `rememberSaveable` as a **note kept in a drawer** (the Android OS).

* **The Mechanism:** It uses `remember` internally to stay fast during recomposition, but it adds a second layer called
  the `SaveableStateRegistry`.
* **The Process:** When the system is about to destroy your Activity (rotation or memory pressure), `rememberSaveable`
  takes your value, converts it into a `Bundle` (a format Android understands), and hands it to the OS.
* **The Restoration:** When the Activity is recreated, Compose asks the OS, "Do you have any saved values for this ID?"
  It then pulls the value back out of the drawer and restores it to the UI.

---

[Understanding state in Jetpack Compose](https://www.youtube.com/watch?v=V-s4z7B_Gnc)
This video explains the fundamental concepts of state management in Compose, including when and why to use different
state-saving APIs.

When you want to use `rememberSaveable` with a custom data class, Android doesn't automatically know how to save it
because the system can only store simple types (like Strings or Ints) in its "storage drawer" (the Bundle).

To fix this, you create a **Saver** to "break down" your object and then "rebuild" it later.

### 1. The Data Class

Imagine you have a simple class that isn't automatically saveable:

```kotlin
data class Person(val name: String, val age: Int)

```

### 2. The Custom Saver

You define a `Saver` object that tells Compose two things: how to **Save** (convert the object to a list or map) and how
to **Restore** (take that list/map and make a `Person` again).

```kotlin
val PersonSaver = Saver<Person, Map<String, Any>>(
    save = { mapOf("name" to it.name, "age" to it.age) }, // How to pack it
    restore = { Person(it["name"] as String, it["age"] as Int) } // How to unpack it
)

```

### 3. How to use it in UI

Pass your custom saver into the `stateSaver` parameter of `rememberSaveable`:

```kotlin
@Composable
fun PersonProfile() {
    var person by rememberSaveable(stateSaver = PersonSaver) {
        mutableStateOf(Person("John", 25))
    }

    // Now if you rotate the screen, 'person' will stay updated! [00:03:38]
}

```

### Why do this?

* **Standard `rememberSaveable**` would crash if you tried to pass it a `Person` object directly because it's not a
  primitive type [[02:11](http://www.youtube.com/watch?v=o4qbOO2XLZU&t=131)].
* **The Saver** acts as a bridge, converting your complex object into a format the Android Bundle can
  handle [[02:46](http://www.youtube.com/watch?v=o4qbOO2XLZU&t=166)].

[Watch: Creating a Custom Saver in Jetpack Compose](http://www.youtube.com/watch?v=o4qbOO2XLZU)

