# SwiftUI Cheat Sheet for Android Engineers

> **Target Audience**: Experienced Android developers transitioning to iOS SwiftUI
> **Last Updated**: December 2024

## Table of Contents
1. [Core Concepts & Mental Model](#core-concepts--mental-model)
2. [Easy: Common Patterns](#easy-common-patterns)
3. [Intermediate: State & Navigation](#intermediate-state--navigation)
4. [Advanced: Performance & Architecture](#advanced-performance--architecture)
5. [Expert: Niche Use Cases](#expert-niche-use-cases)
6. [Learning Resources](#learning-resources)

---

## Core Concepts & Mental Model

### Android → SwiftUI Translation

| Android Concept | SwiftUI Equivalent | Notes |
|----------------|-------------------|-------|
| `Activity` | `App` protocol | Entry point for your app |
| `Fragment` | `View` protocol | Reusable UI component |
| `View` (android.view) | `View` (SwiftUI) | All UI elements conform to View |
| `ViewGroup` | Container views (`VStack`, `HStack`, `ZStack`) | Declarative layout |
| XML layouts | SwiftUI DSL | Pure Swift code |
| `ViewModel` | `@Observable` class or `ObservableObject` | State management |
| `LiveData` | `@Published` properties | Reactive updates |
| `findViewById()` | No equivalent needed | Direct property access |
| `RecyclerView` | `List` or `LazyVStack` | Lazy-loaded collections |
| `ConstraintLayout` | Layout modifiers + GeometryReader | Flexible positioning |
| `Intent` | `NavigationLink` or custom logic | Navigation between screens |
| Jetpack Compose | SwiftUI | Apple's declarative framework |

### Key Differences

**Declarative vs Imperative**: SwiftUI is purely declarative. You describe what the UI should look like for any given state, not how to transition between states.

**Value Types**: SwiftUI Views are structs (value types), not classes. They're recreated frequently—don't worry about performance, SwiftUI optimizes this.

**No IDs Required**: Unlike Android XML, you don't assign IDs. Views are accessed through property wrappers and bindings.

---

## Easy: Common Patterns

### 1. Basic UI Components

#### Text (TextView)
```swift
// Simple text
Text("Hello, World!")

// Styled text
Text("Important")
    .font(.title)
    .foregroundColor(.red)
    .bold()

// Multi-line with alignment
Text("Long text that wraps")
    .multilineTextAlignment(.center)
    .lineLimit(3)
```

#### Button (Button)
```swift
// Basic button
Button("Click Me") {
    print("Tapped")
}

// Custom styled button
Button(action: {
    // Action here
}) {
    Text("Submit")
        .frame(maxWidth: .infinity)
        .padding()
        .background(Color.blue)
        .foregroundColor(.white)
        .cornerRadius(8)
}
```

#### Image (ImageView)
```swift
// System image (SF Symbols)
Image(systemName: "star.fill")
    .resizable()
    .frame(width: 50, height: 50)
    .foregroundColor(.yellow)

// Asset image
Image("myImage")
    .resizable()
    .aspectRatio(contentMode: .fit)
    .frame(width: 200)

// Remote image (requires AsyncImage - iOS 15+)
AsyncImage(url: URL(string: "https://example.com/image.jpg")) { image in
    image.resizable().aspectRatio(contentMode: .fill)
} placeholder: {
    ProgressView()
}
```

#### TextField (EditText)
```swift
@State private var username = ""

TextField("Enter username", text: $username)
    .textFieldStyle(RoundedBorderTextFieldStyle())
    .autocapitalization(.none)

// Secure field (password)
SecureField("Password", text: $password)
```

### 2. Layout Containers

#### VStack (LinearLayout vertical)
```swift
VStack(alignment: .leading, spacing: 16) {
    Text("Title")
    Text("Subtitle")
    Button("Action") { }
}
```

#### HStack (LinearLayout horizontal)
```swift
HStack {
    Image(systemName: "star")
    Text("Featured")
    Spacer() // Pushes content apart
    Text("$9.99")
}
.padding()
```

#### ZStack (FrameLayout)
```swift
ZStack {
    Color.blue // Background
    Text("Overlay")
        .foregroundColor(.white)
}
```

#### ScrollView
```swift
ScrollView {
    VStack(spacing: 20) {
        ForEach(0..<50) { index in
            Text("Row \(index)")
        }
    }
}
```

### 3. Lists (RecyclerView)

#### Basic List
```swift
struct Item: Identifiable {
    let id = UUID()
    let title: String
}

List(items) { item in
    Text(item.title)
}
```

#### List with Sections
```swift
List {
    Section("Section 1") {
        Text("Item 1")
        Text("Item 2")
    }
    Section("Section 2") {
        Text("Item 3")
    }
}
```

#### Custom Row View
```swift
List(items) { item in
    HStack {
        Image(systemName: item.icon)
        VStack(alignment: .leading) {
            Text(item.title)
                .font(.headline)
            Text(item.subtitle)
                .font(.caption)
        }
    }
}
```

### 4. Common Modifiers

```swift
// Padding (margin + padding)
.padding() // All sides
.padding(.horizontal, 20) // Specific sides

// Frame (width/height)
.frame(width: 100, height: 100)
.frame(maxWidth: .infinity) // Match parent

// Background
.background(Color.blue)
.background(
    RoundedRectangle(cornerRadius: 10)
        .fill(Color.gray.opacity(0.2))
)

// Clipping and Shape
.clipShape(RoundedRectangle(cornerRadius: 12))
.clipShape(Circle())

// Shadow
.shadow(radius: 5, x: 0, y: 2)
```

---

## Intermediate: State & Navigation

### 1. State Management (LiveData/ViewModel)

#### @State (Local state like remember in Compose)
```swift
struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") {
                count += 1
            }
        }
    }
}
```

#### @Binding (Two-way binding)
```swift
struct ParentView: View {
    @State private var isOn = false
    
    var body: some View {
        ToggleView(isOn: $isOn) // Pass binding with $
    }
}

struct ToggleView: View {
    @Binding var isOn: Bool
    
    var body: some View {
        Toggle("Switch", isOn: $isOn)
    }
}
```

#### @Observable (iOS 17+) - Preferred for ViewModels
```swift
@Observable
class UserViewModel {
    var username = ""
    var isLoading = false
    
    func fetchUser() {
        isLoading = true
        // Network call...
    }
}

struct ProfileView: View {
    let viewModel = UserViewModel()
    
    var body: some View {
        VStack {
            if viewModel.isLoading {
                ProgressView()
            } else {
                Text(viewModel.username)
            }
        }
    }
}
```

#### ObservableObject (iOS 13+) - Legacy approach
```swift
class UserViewModel: ObservableObject {
    @Published var username = ""
    @Published var isLoading = false
}

struct ProfileView: View {
    @StateObject private var viewModel = UserViewModel()
    // Use @ObservedObject if object is passed from parent
    
    var body: some View {
        Text(viewModel.username)
    }
}
```

### 2. Navigation

#### NavigationStack (iOS 16+)
```swift
NavigationStack {
    List(items) { item in
        NavigationLink(value: item) {
            Text(item.title)
        }
    }
    .navigationDestination(for: Item.self) { item in
        DetailView(item: item)
    }
    .navigationTitle("Items")
}
```

#### NavigationView (Legacy, iOS 13+)
```swift
NavigationView {
    List(items) { item in
        NavigationLink(destination: DetailView(item: item)) {
            Text(item.title)
        }
    }
    .navigationTitle("Items")
}
```

#### Sheet (Modal/Dialog)
```swift
@State private var showingSheet = false

Button("Show Sheet") {
    showingSheet = true
}
.sheet(isPresented: $showingSheet) {
    SheetContentView()
}
```

#### FullScreenCover
```swift
.fullScreenCover(isPresented: $showingFullScreen) {
    FullScreenView()
}
```

#### Alert
```swift
@State private var showingAlert = false

Button("Show Alert") {
    showingAlert = true
}
.alert("Title", isPresented: $showingAlert) {
    Button("OK", role: .cancel) { }
    Button("Delete", role: .destructive) {
        // Delete action
    }
}
```

### 3. Environment & Dependency Injection

#### Environment Values
```swift
@Environment(\.dismiss) var dismiss // Dismiss view
@Environment(\.colorScheme) var colorScheme // Dark/light mode

Button("Close") {
    dismiss()
}
```

#### Custom Environment Objects
```swift
class AppSettings: ObservableObject {
    @Published var theme = "light"
}

// In App or parent view
.environmentObject(AppSettings())

// In child view
@EnvironmentObject var settings: AppSettings
```

### 4. Lifecycle

```swift
.onAppear {
    // Like onCreate or onResume
    viewModel.loadData()
}
.onDisappear {
    // Like onPause or onDestroy
    viewModel.cleanup()
}
.task {
    // Async work, cancelled when view disappears
    await viewModel.fetchData()
}
```

---

## Advanced: Performance & Architecture

### 1. Performance Optimization

#### LazyVStack vs VStack
```swift
// VStack loads all views immediately
VStack {
    ForEach(items) { item in
        RowView(item: item)
    }
}

// LazyVStack loads views on demand (like RecyclerView)
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            RowView(item: item)
        }
    }
}
```

#### Expensive Operations
```swift
// Cache expensive computations
struct ContentView: View {
    let items: [Item]
    
    // Computed once per render
    private var filteredItems: [Item] {
        items.filter { $0.isActive }
    }
    
    var body: some View {
        List(filteredItems) { item in
            Text(item.name)
        }
    }
}
```

#### Equatable for View Diffing
```swift
struct CustomRow: View, Equatable {
    let item: Item
    
    static func == (lhs: CustomRow, rhs: CustomRow) -> Bool {
        lhs.item.id == rhs.item.id
    }
    
    var body: some View {
        Text(item.name)
    }
}

// Use with .equatable()
.equatable()
```

### 2. Custom View Modifiers

```swift
struct CardStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color.white)
            .cornerRadius(10)
            .shadow(radius: 5)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardStyle())
    }
}

// Usage
Text("Hello").cardStyle()
```

### 3. Animations

```swift
@State private var isExpanded = false

Button("Toggle") {
    withAnimation(.spring(response: 0.3)) {
        isExpanded.toggle()
    }
}

Rectangle()
    .frame(height: isExpanded ? 200 : 100)
    .animation(.easeInOut, value: isExpanded)
```

#### Matched Geometry Effect
```swift
@Namespace private var animation

if isExpanded {
    DetailView()
        .matchedGeometryEffect(id: "card", in: animation)
} else {
    ThumbnailView()
        .matchedGeometryEffect(id: "card", in: animation)
}
```

### 4. Gesture Handling

```swift
@State private var offset = CGSize.zero

Circle()
    .offset(offset)
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = value.translation
            }
            .onEnded { _ in
                withAnimation {
                    offset = .zero
                }
            }
    )
```

### 5. Architecture Patterns

#### MVVM with @Observable
```swift
@Observable
class ArticleViewModel {
    var articles: [Article] = []
    var isLoading = false
    var errorMessage: String?
    
    func loadArticles() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            articles = try await apiService.fetchArticles()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

struct ArticleListView: View {
    let viewModel = ArticleViewModel()
    
    var body: some View {
        List(viewModel.articles) { article in
            ArticleRow(article: article)
        }
        .task {
            await viewModel.loadArticles()
        }
    }
}
```

#### Repository Pattern
```swift
protocol ArticleRepository {
    func fetchArticles() async throws -> [Article]
}

class RemoteArticleRepository: ArticleRepository {
    func fetchArticles() async throws -> [Article] {
        // Network call
    }
}

// Inject into ViewModel
@Observable
class ArticleViewModel {
    private let repository: ArticleRepository
    
    init(repository: ArticleRepository) {
        self.repository = repository
    }
}
```

---

## Expert: Niche Use Cases

### 1. Custom Layout with Layout Protocol (iOS 16+)

```swift
struct FlowLayout: Layout {
    var spacing: CGFloat = 8
    
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        // Calculate size based on subviews
        let rows = calculateRows(proposal: proposal, subviews: subviews)
        return CGSize(width: proposal.width ?? 0, height: rows.totalHeight)
    }
    
    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        let rows = calculateRows(proposal: proposal, subviews: subviews)
        var y = bounds.minY
        
        for row in rows.rows {
            var x = bounds.minX
            for index in row {
                let subview = subviews[index]
                let size = subview.sizeThatFits(.unspecified)
                subview.place(at: CGPoint(x: x, y: y), proposal: .unspecified)
                x += size.width + spacing
            }
            y += rows.rowHeight + spacing
        }
    }
}
```

### 2. PreferenceKey (Child-to-Parent Communication)

```swift
struct SizePreferenceKey: PreferenceKey {
    static var defaultValue: CGSize = .zero
    static func reduce(value: inout CGSize, nextValue: () -> CGSize) {
        value = nextValue()
    }
}

struct ChildView: View {
    var body: some View {
        Text("Hello")
            .background(
                GeometryReader { geometry in
                    Color.clear.preference(
                        key: SizePreferenceKey.self,
                        value: geometry.size
                    )
                }
            )
    }
}

struct ParentView: View {
    @State private var childSize: CGSize = .zero
    
    var body: some View {
        ChildView()
            .onPreferenceChange(SizePreferenceKey.self) { size in
                childSize = size
            }
    }
}
```

### 3. ViewBuilder and Result Builders

```swift
@ViewBuilder
func conditionalContent(isLoggedIn: Bool) -> some View {
    if isLoggedIn {
        HomeView()
    } else {
        LoginView()
    }
}

// Custom result builder
@resultBuilder
struct ConditionalViewBuilder {
    static func buildBlock<Content: View>(_ content: Content) -> Content {
        content
    }
    
    static func buildEither<TrueContent: View, FalseContent: View>(
        first: TrueContent
    ) -> _ConditionalContent<TrueContent, FalseContent> {
        .init(storage: .trueContent(first))
    }
}
```

### 4. Custom Transitions

```swift
extension AnyTransition {
    static var customSlide: AnyTransition {
        .asymmetric(
            insertion: .move(edge: .trailing).combined(with: .opacity),
            removal: .move(edge: .leading).combined(with: .opacity)
        )
    }
}

struct ContentView: View {
    @State private var showDetail = false
    
    var body: some View {
        if showDetail {
            DetailView()
                .transition(.customSlide)
        }
    }
}
```

### 5. UIKit Integration (UIViewRepresentable)

```swift
struct MapView: UIViewRepresentable {
    @Binding var region: MKCoordinateRegion
    
    func makeUIView(context: Context) -> MKMapView {
        let mapView = MKMapView()
        mapView.delegate = context.coordinator
        return mapView
    }
    
    func updateUIView(_ mapView: MKMapView, context: Context) {
        mapView.setRegion(region, animated: true)
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, MKMapViewDelegate {
        var parent: MapView
        
        init(_ parent: MapView) {
            self.parent = parent
        }
        
        func mapView(_ mapView: MKMapView, didUpdate userLocation: MKUserLocation) {
            // Handle updates
        }
    }
}
```

### 6. Advanced State Management with Combine

```swift
import Combine

class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var results: [Result] = []
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $searchText
            .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .sink { [weak self] text in
                self?.performSearch(text)
            }
            .store(in: &cancellables)
    }
}
```

### 7. Custom Shape Drawing

```swift
struct Triangle: Shape {
    func path(in rect: CGRect) -> Path {
        var path = Path()
        path.move(to: CGPoint(x: rect.midX, y: rect.minY))
        path.addLine(to: CGPoint(x: rect.maxX, y: rect.maxY))
        path.addLine(to: CGPoint(x: rect.minX, y: rect.maxY))
        path.closeSubpath()
        return path
    }
}

// Animatable shapes
struct AnimatableTriangle: Shape {
    var progress: Double
    
    var animatableData: Double {
        get { progress }
        set { progress = newValue }
    }
    
    func path(in rect: CGRect) -> Path {
        // Draw based on progress
    }
}
```

### 8. Accessibility

```swift
Button(action: { }) {
    Image(systemName: "trash")
}
.accessibilityLabel("Delete item")
.accessibilityHint("Double tap to delete this item permanently")
.accessibilityAddTraits(.isButton)

// Custom accessibility
.accessibilityElement(children: .combine)
.accessibilityValue("\(progress)%")
```

---

## Learning Resources

### Official Documentation
- **Apple SwiftUI Tutorials**: https://developer.apple.com/tutorials/swiftui
- **SwiftUI Documentation**: https://developer.apple.com/documentation/swiftui/
- **Human Interface Guidelines**: https://developer.apple.com/design/human-interface-guidelines/

### Interactive Learning
- **100 Days of SwiftUI** (Free): https://www.hackingwithswift.com/100/swiftui
- **Stanford CS193p** (Free): https://cs193p.sites.stanford.edu/
- **SwiftUI by Example**: https://www.hackingwithswift.com/quick-start/swiftui

### Books
- **Thinking in SwiftUI** by objc.io
- **SwiftUI Apprentice** by Razeware
- **Advanced SwiftUI** by objc.io

### Video Courses
- **Sean Allen YouTube**: Practical SwiftUI tutorials
- **Paul Hudson (Hacking with Swift)**: Comprehensive guides
- **Stewart Lynch YouTube**: Weekly SwiftUI content

### Community & Practice
- **Swift Forums**: https://forums.swift.org/c/related-projects/swiftui/
- **r/SwiftUI**: https://reddit.com/r/SwiftUI/
- **SwiftUI Lab**: https://swiftui-lab.com/
- **Kavsoft YouTube**: Advanced animations and UI

### Tools
- **SF Symbols App**: Browse system icons (free from Apple)
- **Xcode Previews**: Live preview while coding
- **SwiftUI Inspector**: cmd+click any view in Xcode

### Key Differences from Android/Compose
1. **No XML or separate layout files** - everything in Swift
2. **Value semantics** - Views are structs, recreated frequently
3. **Property wrappers** - @State, @Binding, @Observable for reactive state
4. **Modifiers return new views** - chaining creates view hierarchy
5. **Platform integration** - UIKit interop when needed
6. **SF Symbols** - 5000+ built-in vector icons

### Tips for Android Developers
- Think in terms of "source of truth" rather than "view state"
- Embrace value types - don't fight the struct-based approach
- Use Xcode previews extensively - faster than emulator
- SwiftUI is more restrictive than Compose initially, but patterns emerge
- The learning curve is steeper at first, but development is faster once learned

---

**Version**: 1.0 | **Date**: December 2024
**Maintained by**: Community | **Contributions welcome**

