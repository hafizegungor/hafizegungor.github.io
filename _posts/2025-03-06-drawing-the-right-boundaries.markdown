---
layout: post
title: "Drawing the Right Boundaries Between Clean Architecture Layers in Android"
image: 07.jpg
date: 2025-03-06 17:50:18 +0200
tags: [boundaries, clean architecture]
categories: android
---

> After reviewing hundreds of Android codebases, the most common architectural failure isn't using the wrong pattern it's drawing layer boundaries in the wrong place. Here's how to get it right.

Clean Architecture has been a staple of serious Android development for years. Yet I keep seeing the same mistakes: repositories that know about ViewModels, use cases that import Android framework classes, and data models leaking all the way into the UI. The architecture is named correctly, it demands cleanliness and that cleanliness lives or dies at the **boundaries**.

After years of Android development, I've come to believe that most teams don't fail at Clean Architecture because they don't understand the theory. They fail because they draw the boundaries in the wrong places or worse, they draw them correctly on a whiteboard and then violate them in every single file.

This post is a practical, senior-level guide to getting those boundaries right: what they are, why they exist, and how to enforce them in a real Kotlin/Android codebase.

---

## The Dependency Rule is Absolute

**Source code dependencies can only point inward.** Outer layers know about inner layers, never the reverse. Your business logic must be completely unaware that it runs on Android.

```
┌──────────────────────────────────────────────────────────────────┐
│  Framework / UI                                                  │
│  Activities, Fragments, Compose, Room, Retrofit                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Presentation                                              │  │
│  │  ViewModels, UI State, UI Events                           │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │  Data                                                │  │  │
│  │  │  Repository (impl), Data Sources, Mappers            │  │  │
│  │  │  ┌────────────────────────────────────────────────┐  │  │  │
│  │  │  │  Domain  ←  the sacred core                    │  │  │  │
│  │  │  │  Entities, Use Cases, Repository Interfaces    │  │  │  │
│  │  │  └────────────────────────────────────────────────┘  │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

The domain layer defines **interfaces** for everything it needs from the outside world. The data layer implements those interfaces. The presentation layer calls use cases. Nobody skips a layer. Nobody passes data structures from one layer upward without mapping.

---

## The Problem with "Clean Architecture" on Android

Let me be direct: most Android projects that claim to use Clean Architecture are really just three-package MVC with a `UseCase` suffix sprinkled on top.

The symptoms are familiar:

- `UserRepository` returns `LiveData<User>` — an Android framework type leaking into the domain layer
- `GetUserUseCase` imports `android.content.Context`
- `ViewModel` creates `Room` database instances directly
- Data layer models are passed straight into the UI

Each of these is a boundary violation. And each one compounds over time into a codebase that's painful to test, impossible to reuse, and terrifying to change.

---

## The Four Layers, Precisely Defined

Before we talk about boundaries, let's be precise about what each layer actually *owns*.

---

## Layer 1 — The Domain Layer: Your Sacred Core

**Owns:** Entities, value objects, domain-level validation, business rules, repository interfaces, use cases  
**Knows about:** Nothing. Zero dependencies.

The innermost layer. Contains your **pure business objects and rules** things that would be true regardless of whether you're building an Android app, a CLI tool, or a web server.

The domain layer should have **zero Android imports**. Not "almost none." Zero. If you open a file in your `domain` module and see `import android.*` anything, you have a boundary violation.

This layer contains three things.

### 1. Entities — Pure Business Objects

```kotlin
// ✅ Pure Kotlin. No imports from Android, Room, Retrofit, or anything else.
data class User(
    val id: UserId,
    val email: Email,
    val role: UserRole
) {
    fun canAccessAdminPanel(): Boolean = role == UserRole.ADMIN
}

// Wrap primitives with value types where the domain demands it.
@JvmInline
value class UserId(val value: String)

@JvmInline
value class Email(val value: String) {
    init {
        require(value.contains('@')) { "Invalid email: $value" }
    }
}
```

Notice that `Email` validates itself. Business rules live here, not in a `ViewModel` or a `UseCase`.

> **Pro tip:** Wrap primitive types in value classes when the domain assigns meaning to them. A `UserId` and a `PostId` are both strings, but they are not interchangeable. The type system will catch the bug before your users do.

### 2. Repository Interfaces — The Contract, Not the Implementation

```kotlin
// Domain defines what it NEEDS, not how it's provided
interface UserRepository {
    suspend fun getUserById(id: UserId): Result<User>
    suspend fun updateDisplayName(id: UserId, name: String): Result<Unit>
    fun observeUser(id: UserId): Flow<User?>
}
```

> **Common mistake:** Never put `@Inject`, `@Singleton`, Room's `@Entity`, or Retrofit's `@GET` on domain classes. These are framework annotations. The domain doesn't know frameworks exist.

### 3. Use Cases — Single-Responsibility Business Operations

```kotlin
class UpdateUserDisplayNameUseCase @Inject constructor(
    private val userRepository: UserRepository,  // interface, not impl
    private val analyticsRepository: AnalyticsRepository
) {
    suspend operator fun invoke(
        userId: UserId,
        newName: String
    ): Result<Unit> {
        if (newName.isBlank() || newName.length < 2) {
            return Result.failure(InvalidDisplayNameException())
        }
        return userRepository
            .updateDisplayName(userId, newName)
            .onSuccess { analyticsRepository.track(Event.ProfileUpdated) }
    }
}
```

**Rule:** A use case should read like a sentence from the product spec. If you can't describe it in one sentence without saying "and also", split it.

---

## Layer 2 — The Data Layer: Implement, Map, Isolate

**Owns:** Repository implementations, data sources, DTOs, Room entities, API models, mappers  
**Knows about:** Domain interfaces and entities; framework-specific types (Room, Retrofit, etc.) but never leaks them inward

This is where the real work happens and where most boundary violations occur. The data layer implements the domain interfaces. It knows about Room, Retrofit, SharedPreferences, Firebase all of it. But it **never leaks those concepts upward**.

### The Repository Implementation

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
    private val mapper: UserMapper
) : UserRepository {  // ← implements the domain interface

    override suspend fun getUserById(id: UserId): Result<User> {
        return runCatching {
            val dto = remoteDataSource.fetchUser(id.value)
            mapper.toDomain(dto)  // ← always map at the boundary
        }.recoverCatching {
            val entity = localDataSource.getUser(id.value)
                ?: throw UserNotFoundException(id)
            mapper.toDomain(entity)
        }
    }
}
```

> **The mapper is not optional.** Your Room entity and your domain entity will diverge it always happens. Nullable columns, database normalization, caching metadata — these are data-layer concerns. If you skip mappers and use the same model everywhere, you will eventually break your domain because Room needs a column added, or you'll add business logic to a data class that Room manages. Both outcomes are painful.

### Data Source Separation

Split remote and local into separate data sources. The repository orchestrates them; neither data source knows about the other.

```kotlin
// Remote — knows about Retrofit, API models
class UserRemoteDataSource @Inject constructor(
    private val api: UserApi
) {
    suspend fun fetchUser(id: String): UserDto = api.getUser(id)
}

// Local — knows about Room
class UserLocalDataSource @Inject constructor(
    private val dao: UserDao
) {
    suspend fun getUser(id: String): UserEntity? = dao.findById(id)
    suspend fun upsertUser(entity: UserEntity) = dao.upsert(entity)
}
```

---

## Layer 3 — The Presentation Layer: State, Not Logic

**Owns:** ViewModels, UI state models, UI event types, presentation mappers  
**Knows about:** Use cases and domain entities; Android lifecycle types (but doesn't leak them inward)

ViewModels sit at the top of the dependency graph for business logic. They call use cases, transform results into UI state, and expose that state as `StateFlow`. That's it. They do not talk to repositories directly. They do not contain SQL queries. They do not make HTTP calls.

```kotlin
// ✅ ViewModel depends on use cases, never on repositories directly
class UserProfileViewModel @Inject constructor(
    private val getUserProfile: GetUserProfileUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow<UserProfileUiState>(UserProfileUiState.Loading)
    val uiState: StateFlow<UserProfileUiState> = _uiState.asStateFlow()

    fun loadProfile(userId: String) {
        viewModelScope.launch {
            _uiState.value = UserProfileUiState.Loading
            getUserProfile(UserId(userId))
                .onSuccess { profile ->
                    _uiState.value = UserProfileUiState.Success(profile.toUiModel())
                }
                .onFailure { error ->
                    _uiState.value = UserProfileUiState.Error(error.toUiMessage())
                }
        }
    }
}
```

UI models are presentation-layer concerns. Keep them separate from domain entities:

```kotlin
// Presentation mapper — lives in the presentation layer
fun User.toUiModel(): UserProfileUiModel = UserProfileUiModel(
    displayName = "${firstName.uppercase()} $lastName",
    avatarUrl = avatarUrl ?: DEFAULT_AVATAR,
    badgeText = if (role == UserRole.ADMIN) "Admin" else null
)
```

---

## Layer 4 — Frameworks & Drivers (Infrastructure)

**Owns:** `Activity`, `Fragment`, Compose entry points, NavGraph, Hilt modules, database schemas, network configuration  
**Knows about:** Everything (but nothing knows about it)

The outermost layer. Android framework, Room, Retrofit, DataStore all the things that are genuinely hard to test and that change for external reasons.

```kotlin
// ✅ Hilt module wires up the dependency graph in the outermost layer
@Module
@InstallIn(SingletonComponent::class)
object DataModule {

    @Provides
    @Singleton
    fun provideUserRepository(
        userDao: UserDao,
        apiService: UserApiService,
        mapper: UserEntityMapper
    ): UserRepository = UserRepositoryImpl(userDao, apiService, mapper)
}
```

---

## The Dependency Rule

This is the one rule you cannot break:

> **Source code dependencies must point only inward. Nothing in an inner circle can know anything about something in an outer circle.**

Practically, on Android:

| Layer | May import from |
|---|---|
| Entities | Nothing (pure Kotlin stdlib only) |
| Use Cases | Entities, interfaces defined in this layer |
| Interface Adapters | Use Cases, Entities, Android framework types |
| Frameworks & Drivers | Everything |

---

## The Five Most Common Violations (and How to Fix Them)

### 1. Framework types leaking into the domain layer

```kotlin
// ❌ LiveData is an Android type. Domain layer must not know it exists.
interface UserRepository {
    fun getUser(id: String): LiveData<User>
}

// ✅ Use Kotlin coroutines or plain suspend functions instead.
interface UserRepository {
    suspend fun findById(id: UserId): User?
    fun observeUser(id: UserId): Flow<User?>
}
```

`Flow` from `kotlinx.coroutines` is acceptable in the domain layer because it's a pure Kotlin library with no Android dependency. `LiveData` is not it lives in `androidx.lifecycle`.

---

### 2. Data models used as domain models

```kotlin
// ❌ Room entity used directly in the domain layer
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: String,
    @ColumnInfo(name = "email") val email: String,
    @ColumnInfo(name = "role") val role: String  // stringly-typed!
)

// ✅ Separate models, separate concerns
// Domain model (no Room annotations, strongly typed)
data class User(val id: UserId, val email: Email, val role: UserRole)

// Data model (Room-specific)
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: String,
    val email: String,
    val role: String
)

// Mapper lives in the data layer
class UserEntityMapper {
    fun toDomain(entity: UserEntity): User = User(
        id = UserId(entity.id),
        email = Email(entity.email),
        role = UserRole.valueOf(entity.role)
    )
    fun toEntity(domain: User): UserEntity = UserEntity(
        id = domain.id.value,
        email = domain.email.value,
        role = domain.role.name
    )
}
```

Mappers are boilerplate, yes. They are also the only thing standing between a schema migration breaking your domain logic.

---

### 3. Use cases that do too much (or too little)

```kotlin
// ❌ "God" use case — orchestrates, transforms, caches, logs, and formats
class UserUseCase(private val repo: UserRepository) {
    suspend fun getFormattedUserName(id: String): String {
        val user = repo.findById(UserId(id)) ?: throw Exception("Not found")
        analyticsService.log("user_fetched") // Where did this come from?
        return "${user.firstName.uppercase()} ${user.lastName}"  // UI formatting in domain!
    }
}

// ✅ Use case handles one application-level concern
class GetUserUseCase(private val repo: UserRepository) {
    suspend operator fun invoke(id: UserId): Result<User> =
        runCatching { repo.findById(id) ?: throw UserNotFoundException(id) }
}

// Formatting belongs in the presentation layer (a UiModel mapper)
fun User.toUiModel(): UserUiModel = UserUiModel(
    displayName = "${firstName.uppercase()} $lastName"
)
```

---

### 4. ViewModels depending directly on repositories

```kotlin
// ❌ ViewModel bypasses the domain layer entirely
class UserViewModel(private val repo: UserRepositoryImpl) : ViewModel() {
    fun loadUser(id: String) = repo.getUser(id) // Direct Room access, no business logic
}

// ✅ ViewModel depends on a use case, which depends on a repository interface
class UserViewModel(private val getUser: GetUserUseCase) : ViewModel() {
    fun loadUser(id: String) {
        viewModelScope.launch {
            getUser(UserId(id)).onSuccess { /* update state */ }
        }
    }
}
```

This might feel like an extra layer of indirection for simple CRUD. It is. That's the point when the business logic becomes non-trivial (caching strategy, offline-first behavior, permission checks), it belongs in the use case, not scattered across ViewModels.

---

### 5. Context smuggled into the domain layer

```kotlin
// ❌ Context in a use case — untestable and wrong
class GetLocaleUseCase(private val context: Context) {
    operator fun invoke(): Locale = context.resources.configuration.locales[0]
}

// ✅ Define what you need as an interface in the domain layer
interface LocaleProvider {
    fun currentLocale(): Locale
}

class GetLocaleUseCase(private val localeProvider: LocaleProvider) {
    operator fun invoke(): Locale = localeProvider.currentLocale()
}

// Implement it in the framework layer
class AndroidLocaleProvider(private val context: Context) : LocaleProvider {
    override fun currentLocale(): Locale =
        context.resources.configuration.locales[0]
}
```

---

## Enforcing Boundaries with Gradle Modules

The most reliable way to enforce these boundaries is to make them **impossible to violate at compile time**. Structure your project as separate Gradle modules:

```
:app                    → Frameworks & Drivers (depends on all)
:presentation           → ViewModels, UiModels, Mappers (depends on :domain)
:data                   → Repository implementations, DAOs, API services (depends on :domain)
:domain                 → Entities, Use Cases, Repository interfaces (depends on nothing)
```

The dependency graph is explicit and one-directional:

```
:app ──────────────────────────────────────────┐
  │                                            ↓
  ├──→ :presentation ──→ :domain              :domain
  │                                            ↑
  └──→ :data ──────────→ :domain ─────────────┘
```

```kotlin
// domain/build.gradle.kts — zero Android dependencies
plugins {
    id("java-library")
    alias(libs.plugins.kotlin.jvm)
}

dependencies {
    implementation(libs.kotlinx.coroutines.core) // ✅ Pure Kotlin, acceptable
    // No androidx. No Room. No Retrofit. Ever.
}
```

```kotlin
// data/build.gradle.kts
dependencies {
    implementation(project(":domain"))   // knows domain interfaces
    implementation(libs.room.runtime)
    implementation(libs.retrofit)
    // ❌ Cannot accidentally import :presentation — it's not in the graph
}
```

```kotlin
// presentation/build.gradle.kts
dependencies {
    implementation(project(":domain"))   // calls use cases, maps entities to UiModels
    // ❌ Cannot import :data — Room and Retrofit are invisible from here
}
```

When `domain` is a `java-library` module, the Kotlin compiler will refuse to compile any Android import in it. The boundary becomes a **build error**, not a code review comment.

---

## Testing Pays the Dividend

Clean boundaries aren't just an aesthetic choice — they directly determine how testable your code is. When the domain layer has zero Android dependencies, every use case and entity can be tested with plain JUnit, no instrumentation required.

### Testing a Use Case in Pure JUnit

```kotlin
class UpdateUserDisplayNameUseCaseTest {

    private val userRepository: UserRepository = mockk()
    private val analyticsRepository: AnalyticsRepository = mockk(relaxed = true)
    private val useCase = UpdateUserDisplayNameUseCase(userRepository, analyticsRepository)

    @Test
    fun `returns failure when name is blank`() = runTest {
        val result = useCase(UserId("123"), newName = "  ")
        assertTrue(result.isFailure)
        assertIs<InvalidDisplayNameException>(result.exceptionOrNull())
        coVerify(exactly = 0) { userRepository.updateDisplayName(any(), any()) }
    }

    @Test
    fun `tracks analytics event on success`() = runTest {
        coEvery { userRepository.updateDisplayName(any(), any()) } returns Result.success(Unit)
        useCase(UserId("123"), newName = "Jane")
        coVerify { analyticsRepository.track(Event.ProfileUpdated) }
    }
}
```

No `@RunWith(AndroidJUnit4::class)`. No emulator. No Robolectric. Fast, deterministic, runnable on the CI in milliseconds.

### Testing a ViewModel with a Fake Use Case

```kotlin
class UserProfileViewModelTest {

    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()

    @Test
    fun `emits Success state when use case returns a user`() = runTest {
        val fakeUseCase = FakeGetUserProfileUseCase(
            result = Result.success(testUser())
        )
        val viewModel = UserProfileViewModel(fakeUseCase)

        viewModel.loadProfile("123")

        val state = viewModel.uiState.value
        assertIs<UserProfileUiState.Success>(state)
        assertEquals("JANE Doe", state.profile.displayName)
    }

    @Test
    fun `emits Error state when use case fails`() = runTest {
        val fakeUseCase = FakeGetUserProfileUseCase(
            result = Result.failure(UserNotFoundException(UserId("123")))
        )
        val viewModel = UserProfileViewModel(fakeUseCase)

        viewModel.loadProfile("123")

        assertIs<UserProfileUiState.Error>(viewModel.uiState.value)
    }
}
```

Notice that `FakeGetUserProfileUseCase` is a simple in-memory stub — no mocking framework required. This is only possible because the `ViewModel` depends on an interface (the use case), not a concrete implementation.

> **The test surface tells you whether your architecture is healthy.** If your domain tests require `Robolectric` or an emulator, you have a boundary violation somewhere. If your ViewModel tests require a real database, you have bypassed the use case layer.

---

## A Mental Model That Sticks

When you're unsure which layer a class belongs to, ask yourself:

> *"If I wanted to run this project as a command-line tool with no Android SDK — would this class still compile and make sense?"*

- **Yes** → It belongs in the domain layer (`entities` or `use cases`)
- **No, but it would make sense in a Spring Boot web app** → It's an interface adapter (the repository *interface*, a mapper, a DTO)
- **No, it fundamentally requires Android** → It belongs in the frameworks layer

---

## Final Thoughts

Clean Architecture's value isn't that it makes simple things simple. It makes *complex things survivable*. A CRUD screen with one API call doesn't need four layers. But the moment you add offline support, role-based access, analytics, A/B testing, and a second client (a widget, a Wear OS app, a TV variant) — you'll be grateful every boundary is in the right place.

The rules are not bureaucracy. They are the accumulated scar tissue of engineers who shipped the wrong way first.

Draw the lines correctly from day one. Your future self six months from now, staring at a critical bug at 2am will thank you.

---

*If you found this useful, the next logical step is to look at how these boundaries hold up under real pressure: multi-module navigation, shared state across features, and dependency injection at scale.*
