# Jetpack Compose

A comprehensive reference from fundamentals to advanced patterns for experienced Android developers.

---

## Table of Contents
1. [Core Composables & Layouts](#core-composables--layouts)
2. [State Management](#state-management)
3. [Side Effects & Lifecycle](#side-effects--lifecycle)
4. [Modifiers](#modifiers)
5. [Lists & Lazy Layouts](#lists--lazy-layouts)
6. [Navigation](#navigation)
7. [Theming & Styling](#theming--styling)
8. [Performance Optimization](#performance-optimization)
9. [Interop & Migration](#interop--migration)
10. [Advanced Patterns](#advanced-patterns)
11. [Testing](#testing)
12. [Learning Resources](#learning-resources)

---

## Core Composables & Layouts

### Easy: Basic UI Elements

**Text**
```kotlin
Text("Hello World")
Text(
    text = "Styled Text",
    fontSize = 20.sp,
    fontWeight = FontWeight.Bold,
    color = Color.Blue
)
```

**Button**
```kotlin
Button(onClick = { /* action */ }) {
    Text("Click Me")
}
```

**Image**
```kotlin
Image(
    painter = painterResource(R.drawable.icon),
    contentDescription = "Description"
)
```

### Intermediate: Layout Composables

**Column (Vertical)**
```kotlin
Column(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.SpaceEvenly,
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Text("First")
    Text("Second")
}
```

**Row (Horizontal)**
```kotlin
Row(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.SpaceBetween
) {
    Text("Left")
    Text("Right")
}
```

**Box (Stacking)**
```kotlin
Box(modifier = Modifier.size(200.dp)) {
    Image(...)
    Text("Overlay", modifier = Modifier.align(Alignment.BottomEnd))
}
```

### Advanced: Custom Layouts

**Custom Layout**
```kotlin
@Composable
fun CustomLayout(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        val placeables = measurables.map { it.measure(constraints) }
        
        layout(constraints.maxWidth, constraints.maxHeight) {
            var yPosition = 0
            placeables.forEach { placeable ->
                placeable.placeRelative(x = 0, y = yPosition)
                yPosition += placeable.height
            }
        }
    }
}
```

**SubcomposeLayout** (for measuring children conditionally)
```kotlin
@Composable
fun AdaptiveLayout(
    modifier: Modifier = Modifier,
    header: @Composable () -> Unit,
    content: @Composable (headerHeight: Int) -> Unit
) {
    SubcomposeLayout(modifier) { constraints ->
        val headerPlaceable = subcompose("header", header).first().measure(constraints)
        val contentPlaceable = subcompose("content") {
            content(headerPlaceable.height)
        }.first().measure(constraints)
        
        layout(constraints.maxWidth, headerPlaceable.height + contentPlaceable.height) {
            headerPlaceable.place(0, 0)
            contentPlaceable.place(0, headerPlaceable.height)
        }
    }
}
```

---

## State Management

### Easy: Basic State

**remember**
```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
```

**rememberSaveable** (survives configuration changes)
```kotlin
var text by rememberSaveable { mutableStateOf("") }
```

### Intermediate: State Hoisting

```kotlin
@Composable
fun StatefulCounter() {
    var count by remember { mutableStateOf(0) }
    StatelessCounter(
        count = count,
        onIncrement = { count++ }
    )
}

@Composable
fun StatelessCounter(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) {
        Text("Count: $count")
    }
}
```

### Advanced: ViewModel Integration

```kotlin
@Composable
fun MyScreen(viewModel: MyViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    when (uiState) {
        is UiState.Loading -> LoadingScreen()
        is UiState.Success -> SuccessScreen(uiState.data)
        is UiState.Error -> ErrorScreen(uiState.message)
    }
}
```

**StateFlow to State**
```kotlin
val state by viewModel.stateFlow.collectAsState()
```

**LiveData to State**
```kotlin
val state by viewModel.liveData.observeAsState()
```

### Expert: Custom State Management

**Snapshot State**
```kotlin
class MutableStateList<T> {
    private val list = mutableStateListOf<T>()
    
    val snapshot: List<T>
        get() = list.toList()
    
    fun add(item: T) {
        list.add(item)
    }
}
```

**derivedStateOf** (computed state)
```kotlin
val filteredList by remember(searchQuery, items) {
    derivedStateOf {
        items.filter { it.contains(searchQuery) }
    }
}
```

**snapshotFlow** (convert snapshot state to Flow)
```kotlin
LaunchedEffect(Unit) {
    snapshotFlow { scrollState.value }
        .debounce(300)
        .collect { position ->
            // Handle scroll position
        }
}
```

---

## Side Effects & Lifecycle

### Easy: Basic Side Effects

**LaunchedEffect** (runs when key changes)
```kotlin
LaunchedEffect(userId) {
    viewModel.loadUser(userId)
}
```

**DisposableEffect** (cleanup required)
```kotlin
DisposableEffect(Unit) {
    val listener = createListener()
    onDispose {
        listener.remove()
    }
}
```

### Intermediate: Advanced Side Effects

**rememberCoroutineScope**
```kotlin
val scope = rememberCoroutineScope()
Button(onClick = {
    scope.launch {
        // Coroutine code
    }
}) {
    Text("Launch")
}
```

**SideEffect** (sync compose state to non-compose code)
```kotlin
@Composable
fun MyComposable(analyticsService: AnalyticsService) {
    SideEffect {
        analyticsService.setCurrentScreen("MyScreen")
    }
}
```

**rememberUpdatedState** (capture latest value in effect)
```kotlin
@Composable
fun Timer(onTimeout: () -> Unit) {
    val currentOnTimeout by rememberUpdatedState(onTimeout)
    
    LaunchedEffect(Unit) {
        delay(5000)
        currentOnTimeout()
    }
}
```

### Advanced: Effect Lifecycle Control

**produceState** (convert suspend function to State)
```kotlin
@Composable
fun loadNetworkImage(url: String): State<Result<ImageBitmap>> {
    return produceState<Result<ImageBitmap>>(initialValue = Result.Loading, url) {
        value = try {
            Result.Success(loadImage(url))
        } catch (e: Exception) {
            Result.Error(e)
        }
    }
}
```

### Expert: Custom Remember Functions

```kotlin
@Composable
fun <T> rememberMutableStateListOf(vararg elements: T): SnapshotStateList<T> {
    return remember { mutableStateListOf(*elements) }
}

@Composable
inline fun <T> rememberDerivedStateOf(
    crossinline calculation: () -> T
): State<T> = remember { derivedStateOf(calculation) }
```

---

## Modifiers

### Easy: Common Modifiers

```kotlin
Modifier
    .size(100.dp)
    .padding(16.dp)
    .background(Color.Blue)
    .clickable { /* action */ }
```

### Intermediate: Modifier Chains

**Order matters!**
```kotlin
// Padding INSIDE background
Modifier
    .background(Color.Blue)
    .padding(16.dp)

// Padding OUTSIDE background
Modifier
    .padding(16.dp)
    .background(Color.Blue)
```

**Conditional Modifiers**
```kotlin
Modifier.then(
    if (isSelected) Modifier.border(2.dp, Color.Blue)
    else Modifier
)
```

### Advanced: Custom Modifiers

**Simple Modifier Function**
```kotlin
fun Modifier.debugBorder() = this.border(1.dp, Color.Red)
```

**Modifier.composed** (with state)
```kotlin
fun Modifier.shimmer(): Modifier = composed {
    val transition = rememberInfiniteTransition()
    val alpha by transition.animateFloat(
        initialValue = 0.3f,
        targetValue = 1f,
        animationSpec = infiniteRepeatable(tween(1000))
    )
    background(Color.Gray.copy(alpha = alpha))
}
```

### Expert: Modifier.Node API (Compose 1.3+)

```kotlin
class CustomModifierNode : Modifier.Node() {
    override fun ContentDrawScope.draw() {
        // Custom drawing logic
        drawRect(Color.Blue)
        drawContent()
    }
}

fun Modifier.customModifier() = this.then(
    object : ModifierNodeElement<CustomModifierNode>() {
        override fun create() = CustomModifierNode()
        override fun update(node: CustomModifierNode) {}
    }
)
```

**PointerInputModifierNode** (custom gestures)
```kotlin
class DragNode : DelegatingNode(), PointerInputModifierNode {
    override fun onPointerEvent(
        pointerEvent: PointerEvent,
        pass: PointerEventPass,
        bounds: IntSize
    ) {
        // Handle pointer events
    }
}
```

---

## Lists & Lazy Layouts

### Easy: LazyColumn

```kotlin
LazyColumn {
    items(itemsList) { item ->
        ItemRow(item)
    }
}
```

**With Key**
```kotlin
LazyColumn {
    items(itemsList, key = { it.id }) { item ->
        ItemRow(item)
    }
}
```

### Intermediate: LazyRow, LazyVerticalGrid

**LazyRow**
```kotlin
LazyRow(
    contentPadding = PaddingValues(horizontal = 16.dp),
    horizontalArrangement = Arrangement.spacedBy(8.dp)
) {
    items(items) { item ->
        Card { /* content */ }
    }
}
```

**LazyVerticalGrid**
```kotlin
LazyVerticalGrid(
    columns = GridCells.Adaptive(minSize = 128.dp),
    contentPadding = PaddingValues(16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp),
    horizontalArrangement = Arrangement.spacedBy(8.dp)
) {
    items(photos) { photo ->
        PhotoItem(photo)
    }
}
```

### Advanced: Sticky Headers & Item Animations

**Sticky Headers**
```kotlin
LazyColumn {
    groupedItems.forEach { (category, items) ->
        stickyHeader {
            CategoryHeader(category)
        }
        items(items) { item ->
            ItemRow(item)
        }
    }
}
```

**Item Animations**
```kotlin
LazyColumn {
    items(
        items = itemsList,
        key = { it.id }
    ) { item ->
        ItemRow(
            item = item,
            modifier = Modifier.animateItemPlacement()
        )
    }
}
```

### Expert: Custom Lazy Layout

```kotlin
@Composable
fun LazyCustomLayout(
    items: List<Item>,
    modifier: Modifier = Modifier
) {
    val state = rememberLazyListState()
    
    LazyLayout(
        items = items,
        modifier = modifier,
        prefetchState = state.prefetchState
    ) {
        // Custom layout logic
    }
}
```

**Paging Integration**
```kotlin
@Composable
fun PagingList(viewModel: MyViewModel) {
    val items = viewModel.itemsFlow.collectAsLazyPagingItems()
    
    LazyColumn {
        items(
            count = items.itemCount,
            key = items.itemKey { it.id }
        ) { index ->
            items[index]?.let { item ->
                ItemRow(item)
            }
        }
        
        item {
            when (items.loadState.append) {
                is LoadState.Loading -> LoadingItem()
                is LoadState.Error -> ErrorItem()
                else -> {}
            }
        }
    }
}
```

---

## Navigation

### Easy: Basic Navigation

```kotlin
val navController = rememberNavController()

NavHost(navController, startDestination = "home") {
    composable("home") { HomeScreen(navController) }
    composable("details/{itemId}") { backStackEntry ->
        DetailsScreen(backStackEntry.arguments?.getString("itemId"))
    }
}

// Navigate
navController.navigate("details/123")
```

### Intermediate: Type-Safe Navigation

```kotlin
// Define routes
sealed class Screen(val route: String) {
    object Home : Screen("home")
    data class Details(val id: String) : Screen("details/$id") {
        companion object {
            const val route = "details/{itemId}"
        }
    }
}

// Setup
composable(Screen.Details.route) { backStackEntry ->
    val itemId = backStackEntry.arguments?.getString("itemId")
    DetailsScreen(itemId)
}

// Navigate
navController.navigate(Screen.Details("123").route)
```

### Advanced: Bottom Navigation & Nested Graphs

```kotlin
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    Scaffold(
        bottomBar = {
            BottomNavigation {
                BottomNavigationItem(
                    selected = currentRoute == "home",
                    onClick = { navController.navigate("home") },
                    icon = { Icon(Icons.Default.Home) }
                )
            }
        }
    ) {
        NavHost(navController, "home") {
            navigation(startDestination = "feed", route = "home") {
                composable("feed") { FeedScreen() }
                composable("profile") { ProfileScreen() }
            }
            composable("settings") { SettingsScreen() }
        }
    }
}
```

### Expert: Deep Linking & Result Handling

**Deep Links**
```kotlin
composable(
    route = "details/{itemId}",
    deepLinks = listOf(navDeepLink {
        uriPattern = "myapp://details/{itemId}"
    })
) { backStackEntry ->
    DetailsScreen(backStackEntry.arguments?.getString("itemId"))
}
```

**Navigation Result**
```kotlin
// Set result
navController.previousBackStackEntry
    ?.savedStateHandle
    ?.set("key", result)
navController.popBackStack()

// Get result
val result = navController.currentBackStackEntry
    ?.savedStateHandle
    ?.getStateFlow<String?>("key", null)
    ?.collectAsState()
```

---

## Theming & Styling

### Easy: Material Theme

```kotlin
@Composable
fun MyApp() {
    MaterialTheme(
        colorScheme = lightColorScheme(
            primary = Color(0xFF6200EE),
            secondary = Color(0xFF03DAC6)
        ),
        typography = Typography(
            bodyLarge = TextStyle(fontSize = 16.sp)
        )
    ) {
        // App content
    }
}
```

### Intermediate: Dynamic Theming

```kotlin
@Composable
fun MyApp() {
    val darkTheme = isSystemInDarkTheme()
    val colorScheme = if (darkTheme) darkColorScheme() else lightColorScheme()
    
    MaterialTheme(colorScheme = colorScheme) {
        Surface(color = MaterialTheme.colorScheme.background) {
            // Content
        }
    }
}
```

### Advanced: Custom Theme Extensions

```kotlin
data class ExtendedColors(
    val customColor: Color,
    val anotherColor: Color
)

val LocalExtendedColors = staticCompositionLocalOf {
    ExtendedColors(
        customColor = Color.Unspecified,
        anotherColor = Color.Unspecified
    )
}

@Composable
fun ExtendedTheme(
    content: @Composable () -> Unit
) {
    val extendedColors = ExtendedColors(
        customColor = Color(0xFF123456),
        anotherColor = Color(0xFF654321)
    )
    
    CompositionLocalProvider(LocalExtendedColors provides extendedColors) {
        MaterialTheme {
            content()
        }
    }
}

// Usage
val customColor = LocalExtendedColors.current.customColor
```

### Expert: Custom Design System

```kotlin
object MyDesignSystem {
    val colors = MyColors()
    val typography = MyTypography()
    val shapes = MyShapes()
}

@Composable
fun MyTheme(content: @Composable () -> Unit) {
    CompositionLocalProvider(
        LocalMyColors provides MyDesignSystem.colors,
        LocalMyTypography provides MyDesignSystem.typography
    ) {
        content()
    }
}

object MyTheme {
    val colors: MyColors
        @Composable
        get() = LocalMyColors.current
    
    val typography: MyTypography
        @Composable
        get() = LocalMyTypography.current
}
```

---

## Performance Optimization

### Easy: Key Optimizations

**Avoid Unnecessary Recompositions**
```kotlin
// Bad - recomposes on any state change
@Composable
fun BadExample(viewModel: MyViewModel) {
    val state = viewModel.state
    Text(state.user.name)
}

// Good - only recomposes when name changes
@Composable
fun GoodExample(viewModel: MyViewModel) {
    val userName by remember { derivedStateOf { viewModel.state.user.name } }
    Text(userName)
}
```

### Intermediate: Stability & Skipping

**Stable Classes**
```kotlin
@Immutable
data class User(val id: String, val name: String)

@Stable
class UserRepository {
    fun getUser(id: String): User = TODO()
}
```

**Remember Lambdas**
```kotlin
// Bad - creates new lambda on every recomposition
Button(onClick = { viewModel.doSomething() }) {
    Text("Click")
}

// Good - stable reference
val onClick = remember { { viewModel.doSomething() } }
Button(onClick = onClick) {
    Text("Click")
}

// Better - if viewModel is stable
Button(onClick = viewModel::doSomething) {
    Text("Click")
}
```

### Advanced: Defer Reads & Keys

**Defer State Reads**
```kotlin
// Read state as late as possible
@Composable
fun ItemList(items: List<Item>) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            // State read happens here, not at LazyColumn level
            ItemRow(item)
        }
    }
}
```

**Proper Keys**
```kotlin
// Maintains scroll position and animations
LazyColumn {
    items(
        items = itemsList,
        key = { it.id }  // Stable, unique identifier
    ) { item ->
        ItemRow(item)
    }
}
```

### Expert: Composition Local & Baseline Profiles

**Efficient CompositionLocal**
```kotlin
// Prefer staticCompositionLocalOf for rarely changing values
val LocalUserId = staticCompositionLocalOf { "" }

// Use compositionLocalOf for frequently changing values
val LocalCurrentTime = compositionLocalOf { 0L }
```

**Baseline Profiles** (in baseline-prof.txt)
```
HSPLcom/example/MyComposable;->MyComposable(Landroidx/compose/runtime/Composer;I)V
HSPLcom/example/MyScreen;->MyScreen(Landroidx/compose/runtime/Composer;I)V
```

**Recomposition Debugging**
```kotlin
@Composable
fun LogCompositions(tag: String) {
    SideEffect {
        Log.d("Composition", "$tag recomposed")
    }
}
```

---

## Interop & Migration

### Easy: Compose in Views

```kotlin
class MyActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp()
        }
    }
}

// In Fragment
override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, bundle: Bundle?) =
    ComposeView(requireContext()).apply {
        setContent {
            MyScreen()
        }
    }
```

### Intermediate: Views in Compose

**AndroidView**
```kotlin
@Composable
fun CustomView() {
    AndroidView(
        factory = { context ->
            MyCustomView(context).apply {
                // Initialize view
            }
        },
        update = { view ->
            // Update view when state changes
            view.updateData(data)
        }
    )
}
```

### Advanced: Interop Lifecycle

**ViewModel Sharing**
```kotlin
// In Activity
val sharedViewModel: SharedViewModel by viewModels()

setContent {
    // Composable gets same instance
    MyScreen(viewModel = sharedViewModel)
}
```

**Fragment to Compose Communication**
```kotlin
@Composable
fun FragmentContainer(fragmentManager: FragmentManager) {
    AndroidViewBinding(FragmentContainerBinding::inflate) { binding ->
        fragmentManager.beginTransaction()
            .replace(binding.fragmentContainer.id, MyFragment())
            .commit()
    }
}
```

### Expert: Incremental Migration

**Mixed Hierarchies**
```kotlin
// Compose -> View -> Compose
@Composable
fun MixedScreen() {
    Column {
        Text("Compose Top")
        
        AndroidView(
            factory = { context ->
                ComposeView(context).apply {
                    setContent {
                        Text("Nested Compose")
                    }
                }
            }
        )
        
        Text("Compose Bottom")
    }
}
```

---

## Advanced Patterns

### Advanced: Remember with Multiple Keys

```kotlin
val result = remember(key1, key2, key3) {
    expensiveOperation(key1, key2, key3)
}
```

### Advanced: Slot APIs

```kotlin
@Composable
fun MyCard(
    title: @Composable () -> Unit,
    content: @Composable () -> Unit,
    actions: @Composable RowScope.() -> Unit
) {
    Card {
        Column {
            title()
            content()
            Row { actions() }
        }
    }
}

// Usage
MyCard(
    title = { Text("Title") },
    content = { Text("Content") },
    actions = {
        Button(onClick = {}) { Text("OK") }
        Button(onClick = {}) { Text("Cancel") }
    }
)
```

### Expert: CompositionLocal Patterns

```kotlin
data class AppState(val user: User, val settings: Settings)

val LocalAppState = staticCompositionLocalOf<AppState> {
    error("No AppState provided")
}

@Composable
fun ProvideAppState(
    appState: AppState,
    content: @Composable () -> Unit
) {
    CompositionLocalProvider(LocalAppState provides appState) {
        content()
    }
}

// Deep in hierarchy
@Composable
fun DeepComponent() {
    val appState = LocalAppState.current
    Text("User: ${appState.user.name}")
}
```

### Expert: Recomposition Optimization with Snapshot State

```kotlin
@Composable
fun OptimizedList() {
    val items = remember { mutableStateListOf<Item>() }
    
    // Only items that changed will recompose
    LazyColumn {
        items(items, key = { it.id }) { item ->
            ItemRow(item)
        }
    }
    
    // Modifying list triggers minimal recomposition
    Button(onClick = { items[0] = items[0].copy(name = "New Name") }) {
        Text("Update First")
    }
}
```

### Expert: Custom Layout Animations

```kotlin
@Composable
fun AnimatedLayout(
    targetState: LayoutState,
    content: @Composable (LayoutState) -> Unit
) {
    val transition = updateTransition(targetState)
    
    transition.AnimatedContent(
        transitionSpec = {
            fadeIn() + slideInHorizontally() with
            fadeOut() + slideOutHorizontally()
        }
    ) { state ->
        content(state)
    }
}
```

---

## Testing

### Easy: Basic Tests

```kotlin
@Test
fun myTest() {
    composeTestRule.setContent {
        MyComposable()
    }
    
    composeTestRule.onNodeWithText("Hello").assertIsDisplayed()
}
```

### Intermediate: Finders & Actions

```kotlin
@Test
fun testInteraction() {
    composeTestRule.setContent { MyScreen() }
    
    // Find and click
    composeTestRule.onNodeWithText("Button").performClick()
    
    // Find by content description
    composeTestRule.onNodeWithContentDescription("icon").assertExists()
    
    // Find by tag
    composeTestRule.onNodeWithTag("my-tag").assertIsDisplayed()
    
    // Type text
    composeTestRule.onNodeWithTag("input").performTextInput("Hello")
}
```

### Advanced: Semantics & Custom Matchers

```kotlin
// Add semantics
Text(
    "Hello",
    modifier = Modifier.semantics {
        testTag = "greeting"
        contentDescription = "Greeting text"
    }
)

// Custom matcher
fun hasClickAction(): SemanticsMatcher =
    SemanticsMatcher.expectValue(SemanticsActions.OnClick, null)

@Test
fun testCustomMatcher() {
    composeTestRule.onNode(hasClickAction()).performClick()
}
```

### Expert: Testing State & Time

```kotlin
@Test
fun testAsyncState() = runTest {
    val viewModel = MyViewModel()
    
    composeTestRule.setContent {
        MyScreen(viewModel)
    }
    
    // Advance time
    composeTestRule.mainClock.advanceTimeBy(1000L)
    
    // Wait for condition
    composeTestRule.waitUntil(5000L) {
        composeTestRule.onAllNodesWithText("Loaded")
            .fetchSemanticsNodes().isNotEmpty()
    }
    
    composeTestRule.onNodeWithText("Loaded").assertIsDisplayed()
}
```

**Testing Side Effects**
```kotlin
@Test
fun testLaunchedEffect() {
    var effectExecuted = false
    
    composeTestRule.setContent {
        LaunchedEffect(Unit) {
            delay(100)
            effectExecuted = true
        }
    }
    
    composeTestRule.mainClock.advanceTimeBy(100)
    composeTestRule.waitForIdle()
    
    assertTrue(effectExecuted)
}
```

---

## Learning Resources

### Official Documentation
- **Compose Docs**: https://developer.android.com/jetpack/compose
- **Compose Samples**: https://github.com/android/compose-samples
- **API Reference**: https://developer.android.com/reference/kotlin/androidx/compose/package-summary
- **Compose Pathway**: https://developer.android.com/courses/pathways/compose

### Deep Dives
- **Thinking in Compose**: https://developer.android.com/jetpack/compose/mental-model
- **Lifecycle**: https://developer.android.com/jetpack/compose/lifecycle
- **State Management**: https://developer.android.com/jetpack/compose/state
- **Performance**: https://developer.android.com/jetpack/compose/performance

### Video Content
- **Android Developers YouTube**: Compose tutorials and talks
- **Conference Talks**: Android Dev Summit, Google I/O
- **Compose Camp**: Structured learning program

### Advanced Topics
- **Compose Compiler**: Understanding how Compose works under the hood
- **Snapshot System**: Deep dive into state management internals
- **Modifier System**: Understanding the modifier chain
- **Layout Inspector**: Debugging compose hierarchies

### Community Resources
- **r/android_devs**: Reddit community
- **Kotlin Slack**: #compose channel
- **Stack Overflow**: [android-jetpack-compose] tag

### Books & Courses
- "Jetpack Compose Internals" by Jorge Castillo
- Android Developer courses on Compose
- Pluralsight and Udemy Compose courses

### Blogs & Articles
- **Chris Banes' Blog**: Compose deep dives
- **Android Developers Blog**: Official updates
- **Medium**: Tag search for "jetpack-compose"

### Tools
- **Compose Preview**: Live preview in Android Studio
- **Layout Inspector**: Debug compose layouts
- **Recomposition Highlighter**: Visual debugging tool
- **Baseline Profile Generator**: Performance optimization

---

## Quick Reference: Common Patterns

**Loading State**
```kotlin
when (uiState) {
    is Loading -> CircularProgressIndicator()
    is Success -> Content(uiState.data)
    is Error -> ErrorMessage(uiState.error)
}
```

**Pull to Refresh**
```kotlin
val refreshing by viewModel.refreshing.collectAsState()
PullRefreshIndicator(refreshing, viewModel::refresh)
```

**Dialog**
```kotlin
if (showDialog) {
    AlertDialog(
        onDismissRequest = { showDialog = false },
        title = { Text("Title") },
        text = { Text("Message") },
        confirmButton = {
            TextButton(onClick = { showDialog = false }) {
                Text("OK")
            }
        }
    )
}
```

**Scaffold with Top Bar**
```kotlin
Scaffold(
    topBar = {
        TopAppBar(title = { Text("Title") })
    }
) { paddingValues ->
    Content(Modifier.padding(paddingValues))
}
```

---
