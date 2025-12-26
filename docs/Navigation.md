In November 2025, Google officially released **Navigation 3 (Nav3)** as a stable library. It is a fundamental shift from
the "current" Navigation Compose (Nav2) you are likely using.

The main difference is that **Nav2** treats navigation as a "black box" where the library owns the state, while **Nav3**
treats navigation as a **State-first** list that **you** own.

---

### 1. The Philosophical Difference

| Feature          | Navigation Compose (Current/Nav2)                     | Navigation 3 (New/Nav3)                                        |
|------------------|-------------------------------------------------------|----------------------------------------------------------------|
| **Ownership**    | The library owns the backstack (via `NavController`). | **You** own the backstack (as a `SnapshotStateList`).          |
| **Routing**      | String-based or Type-safe routes.                     | **Keys-based**. A screen is just a key in a list.              |
| **UI Component** | `NavHost`                                             | `NavDisplay`                                                   |
| **Philosophy**   | Command-based (`navController.navigate()`).           | **Declarative**. You add an item to a list, and the UI reacts. |

---

### 2. Deep Dive: Key Technical Differences

#### A. The Backstack is just a `List`

In the current library, you can’t easily see or modify the entire backstack without hacks. In **Nav3**, the backstack is
literally just a list of "keys" (serializable classes).

* **Nav2:** You call `Maps("details")` and trust the library to add it to its internal hidden stack.
* **Nav3:** You have a `mutableStateListOf(HomeKey, DetailsKey)`. To go back, you simply call `list.removeLast()`.

#### B. The "Speed Limit" Tech for Devs: Adaptive Layouts

One of the biggest pain points in the current library is **List-Detail** views (showing two screens side-by-side on a
tablet).

* **Current:** You often have to manage two separate `NavControllers` or manually swap composables.
* **Nav3:** It introduces **Scenes**. A `NavDisplay` can use a `TwoPaneSceneStrategy`. It looks at your list, sees two
  keys, and decides to show them side-by-side automatically because the screen is wide enough.

#### C. Handling the Speed Limit (Road Sign Analogy)

In your previous question, you liked the "one-button" sync in the Lexus. Nav3 has a similar concept for state:

* In the current library, if you rotate the screen or the process dies, the `NavController` tries to "reconstruct" the
  state from its internal logic.
* In **Nav3**, because the state is just a list in your `ViewModel` or `SavedStateHandle`, it is **"Transparent."**There
  is no guessing; the UI is always a direct reflection of your data.

---

### 3. Comparison of Code

**Current Navigation (Nav2):**

```kotlin
// You define a graph and the library manages the stack
NavHost(navController, startDestination = "home") {
    composable("home") { HomeScreen(onNavigate = { navController.navigate("details") }) }
    composable("details") { DetailsScreen() }
}

```

**Navigation 3 (Nav3):**

```kotlin
// You own the list of screens
val backStack = rememberNavBackStack(HomeKey)

NavDisplay(
    backStack = backStack,
    entryProvider = entryProvider {
        entry<HomeKey> { HomeScreen(onNavigate = { backStack.add(DetailsKey) }) }
        entry<DetailsKey> { DetailsScreen() }
    }
)

```
