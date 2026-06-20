ShopApp — Application E-Commerce Android

# 🛒 ShopApp — Android E-Commerce Application

> A complete e-commerce mobile application built in **Android Kotlin** with **MVVM** architecture, **Room**, **Navigation Component**, **StateFlow**, and **Material Design 3**.

---

## 📱 Overview

ShopApp is an Android mobile application that lets users browse a product catalog, add items to their cart, and manage a list of favorites. It demonstrates modern Android development best practices by combining a robust architecture with a reactive user interface.

---

## ✨ Features

- 🔐 **Authentication** — Secure login with session management via SharedPreferences
- 🗂️ **Product catalog** — Grid display using RecyclerView + GridLayoutManager
- 🔍 **Product detail** — Full view with description, price, and actions
- 🛒 **Cart** — Add/remove items with a real-time notification badge
- ❤️ **Favorites** — Personalized list persisted locally
- 🔄 **Synchronization** — Local data (Room) and remote data (REST API)

---

## 🏗️ Architecture

The project follows the **MVVM (Model-View-ViewModel)** pattern recommended by Google, with a clear separation of responsibilities.

```
app/
├── data/
│   ├── local/
│   │   ├── dao/          # Room DAOs (CartDao, FavoriteDao, ProductDao)
│   │   ├── entity/       # Room Entities (Product, CartItem, Favorite)
│   │   └── AppDatabase   # Room Database
│   ├── remote/
│   │   ├── api/          # Retrofit interface (ProductApiService)
│   │   └── dto/          # Data Transfer Objects
│   └── repository/       # Repository (single source of truth)
│       └── ProductRepository
├── ui/
│   ├── catalog/          # Catalog Fragment + ViewModel
│   ├── detail/           # Product detail Fragment
│   ├── cart/             # Cart Fragment + ViewModel
│   ├── favorites/        # Favorites Fragment
│   └── shared/           # SharedViewModel (cart badge)
├── utils/
│   ├── SessionManager    # SharedPreferences management
│   └── DiffCallback      # DiffUtil for RecyclerView
└── navigation/
    └── nav_graph.xml     # Navigation graph
```

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-------------|
| Language | Kotlin |
| Architecture | MVVM + Repository Pattern |
| Local database | Room (SQLite) |
| Networking | Retrofit 2 + OkHttp |
| Navigation | Navigation Component (Jetpack) |
| Reactivity | LiveData & StateFlow |
| Dependency injection | Hilt / Manual DI |
| UI | Material Design 3, RecyclerView, GridLayoutManager |
| Session persistence | SharedPreferences |
| Image loading | Glide / Coil |
| Async | Kotlin Coroutines |

---

## 🗄️ Room Database Schema

```kotlin
// Product Entity
@Entity(tableName = "products")
data class Product(
    @PrimaryKey val id: Int,
    val title: String,
    val price: Double,
    val description: String,
    val imageUrl: String,
    val category: String
)

// Cart Entity
@Entity(
    tableName = "cart_items",
    foreignKeys = [ForeignKey(
        entity = Product::class,
        parentColumns = ["id"],
        childColumns = ["productId"],
        onDelete = ForeignKey.CASCADE
    )]
)
data class CartItem(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val productId: Int,
    val quantity: Int
)

// Favorites Entity
@Entity(
    tableName = "favorites",
    foreignKeys = [ForeignKey(
        entity = Product::class,
        parentColumns = ["id"],
        childColumns = ["productId"],
        onDelete = ForeignKey.CASCADE
    )]
)
data class Favorite(
    @PrimaryKey val productId: Int
)
```

---

## 🔄 Data Flow

```
REST API ──────────────────────────────────┐
                                           ▼
Room (local) ────────► Repository ────► ViewModel ────► Fragment/Activity
                           │                │
                    Single source       StateFlow /
                    of truth            LiveData
```

The **Repository** centralizes all data sources. It decides whether to use the local cache (Room) or make a network call (Retrofit) depending on data availability and freshness.

---

## 📐 Navigation

The navigation graph (Navigation Component) defines the transitions between screens:

```
LoginFragment
     │
     ▼
CatalogFragment ──────► ProductDetailFragment
     │                          │
     ▼                          ▼
FavoritesFragment          CartFragment
     └──────────────────────────┘
```

Arguments between fragments are passed via **Safe Args**, ensuring type safety.

---

## ⚡ Reactivity with StateFlow & LiveData

```kotlin
// In CartViewModel
private val _cartItems = MutableStateFlow<List<CartItem>>(emptyList())
val cartItems: StateFlow<List<CartItem>> = _cartItems.asStateFlow()

val cartBadgeCount: StateFlow<Int> = cartItems
    .map { it.sumOf { item -> item.quantity } }
    .stateIn(viewModelScope, SharingStarted.Lazily, 0)

// Observation in the Fragment
lifecycleScope.launch {
    viewModel.cartBadgeCount.collect { count ->
        updateCartBadge(count)
    }
}
```

---

## 🔐 Session Management (SharedPreferences)

```kotlin
class SessionManager(context: Context) {
    private val prefs = context.getSharedPreferences("user_session", Context.MODE_PRIVATE)

    fun saveAuthToken(token: String) = prefs.edit().putString("auth_token", token).apply()
    fun getAuthToken(): String? = prefs.getString("auth_token", null)
    fun isLoggedIn(): Boolean = getAuthToken() != null
    fun clearSession() = prefs.edit().clear().apply()
}
```

---

## 📋 Grading Rubric & Implementation

| Question | Points | Status |
|----------|--------|--------|
| MVVM architecture with Repository, Room DAO, and shared ViewModel | 3 pts | ✅ |
| Room entities with relationships (@Entity, @ForeignKey) | 3 pts | ✅ |
| RecyclerView with GridLayoutManager, DiffUtil, and Adapter | 3 pts | ✅ |
| LiveData/StateFlow for the dynamic cart badge | 3 pts | ✅ |
| Navigation between fragments with Navigation Component | 3 pts | ✅ |
| Room persistence (transactions, migrations) | 3 pts | ✅ |
| SharedPreferences for session and tokens | 2 pts | ✅ |
| **Total** | **20 pts** | |

---

## 🚀 Installation & Launch

### Prerequisites

- Android Studio Hedgehog (2023.1.1) or higher
- JDK 17+
- Android SDK 24+ (minSdk 24)
- Gradle 8.x

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/shopapp-android.git
cd shopapp-android

# 2. Open in Android Studio
# File > Open > select the project folder

# 3. Sync Gradle
# Android Studio does this automatically, or: ./gradlew sync

# 4. Run the application
# Run > Run 'app' (Shift+F10)
```

### API Configuration

Add your base URL in `local.properties`:

```properties
BASE_URL="https://fakestoreapi.com/"
```

Then in `build.gradle (app)`:

```kotlin
buildConfigField("String", "BASE_URL", localProperties["BASE_URL"] as String)
```

---

## 📦 Main Dependencies

```kotlin
// build.gradle (app)
dependencies {
    // Architecture
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")

    // Room
    implementation("androidx.room:room-runtime:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    kapt("androidx.room:room-compiler:2.6.1")

    // Navigation
    implementation("androidx.navigation:navigation-fragment-ktx:2.7.7")
    implementation("androidx.navigation:navigation-ui-ktx:2.7.7")

    // Networking
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")

    // UI
    implementation("com.google.android.material:material:1.11.0")
    implementation("io.coil-kt:coil:2.6.0")

    // Hilt (DI)
    implementation("com.google.dagger:hilt-android:2.51")
    kapt("com.google.dagger:hilt-compiler:2.51")
}
```

---

## 📸 Screenshots

> *(To be added after the application is deployed)*

| Catalog | Product Detail | Cart | Favorites |
|-----------|---------------|--------|---------|
| ![Catalog](screenshots/catalog.png) | ![Detail](screenshots/detail.png) | ![Cart](screenshots/cart.png) | ![Favorites](screenshots/favorites.png) |

---

## 🤝 Contributors

| Name | Role |
|-----|------|
| *(Your name)* | Lead Developer |

---

## 📄 License

This project was created as part of an academic project. Any reproduction without authorization is prohibited.

---

<p align="center">
  Built with ❤️ in <strong>Kotlin</strong> · Android Jetpack · MVVM
</p>
