# 🧾 Jetpack Compose Cheat Sheet

https://developer.android.com/compose

## 🏗️ Basic Structure
```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello, $name!")
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    Greeting("Android")
}
```

---

## 🧱 Layouts

| Type         | Usage Example                  |
|--------------|--------------------------------|
| `Column`     | `Column { ... }`               |
| `Row`        | `Row { ... }`                  |
| `Box`        | `Box { ... }`                  |
| `LazyColumn` | Scrollable list                |
| `LazyRow`    | Horizontal list                |

### Modifiers
```kotlin
Modifier
    .padding(16.dp)
    .fillMaxWidth()
    .background(Color.Gray)
    .clickable { ... }
```

---

## 🎨 UI Elements

| Element           | Example                                        |
|------------------|------------------------------------------------|
| `Text`           | `Text("Hello")`                                |
| `Button`         | `Button(onClick = { ... }) { Text("Click") }`  |
| `Icon`           | `Icon(Icons.Default.Favorite, null)`           |
| `Image`          | `Image(painter, contentDescription)`           |
| `TextField`      | `OutlinedTextField(value, onValueChange)`      |

---

## 🧠 State Management

### `remember` and `mutableStateOf`
```kotlin
val count = remember { mutableStateOf(0) }
Text("Count: ${count.value}")
Button(onClick = { count.value++ }) { Text("Increment") }
```

### `rememberSaveable`
```kotlin
val name by rememberSaveable { mutableStateOf("") }
```

### ViewModel + StateFlow
```kotlin
val uiState by viewModel.uiState.collectAsState()
```

---

## 🧩 Effects

| Hook                  | Use Case                        |
|-----------------------|----------------------------------|
| `LaunchedEffect`      | Run suspend function on key      |
| `rememberUpdatedState`| Use latest lambda in callbacks   |
| `DisposableEffect`    | Lifecycle cleanup (onDispose)    |
| `SideEffect`          | React to successful recomposition|

### Example: `LaunchedEffect`
```kotlin
LaunchedEffect(Unit) {
    viewModel.loadData()
}
```

---

## 📦 Navigation
```kotlin
val navController = rememberNavController()
NavHost(navController, startDestination = "home") {
    composable("home") { HomeScreen(navController) }
    composable("details/{id}") { backStack ->
        val id = backStack.arguments?.getString("id")
        DetailScreen(id)
    }
}
```

---

## 🖼️ Themes & Styling
```kotlin
MaterialTheme(
    typography = myTypography,
    colors = myColorPalette,
    shapes = myShapes
) {
    // App UI
}
```
Use `MaterialTheme.typography.bodyLarge` etc.

---

## 🧪 UI Testing
```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun myTest() {
    composeTestRule.setContent {
        MyComposable()
    }

    composeTestRule
        .onNodeWithText("Click me")
        .assertExists()
        .performClick()
}
```
Use `Modifier.testTag("tag")` to help target nodes.

---

## 🪄 Animations
```kotlin
val visible = remember { mutableStateOf(true) }

AnimatedVisibility(visible = visible.value) {
    Text("Hello Compose!")
}
```
Other APIs: `animate*AsState`, `Crossfade`, `updateTransition`

---

## 📱 XML Integration

### In XML Layout:
```xml
<androidx.compose.ui.platform.ComposeView
    android:id="@+id/compose_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"/>
```

### In Kotlin:
```kotlin
composeView.setContent {
    MaterialTheme {
        MyComposable()
    }
}
```

---

## 🧩 Best Practices

| Pattern           | Usage                                     |
|------------------|--------------------------------------------|
| State Hoisting   | Pass value + onValueChange to parent       |
| Slot API         | Custom Composables with content lambdas    |
| Preview Params   | Provide sample data for @Preview           |
| ViewModel Split  | Logic in ViewModel, UI in Composable       |

---

## 🧰 Helpful Libraries

| Library                  | Purpose                              |
|--------------------------|--------------------------------------|
| `Coil-Compose`           | Load images in Compose               |
| `Accompanist`            | Pager, insets, permissions, etc.     |
| `Showkase`               | UI catalog & visual preview registry |
| `Compose Destinations`   | Safe navigation                      |
| `Material3`              | Material You components              |
