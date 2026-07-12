MainActivity.kt - الشاشة الرئيسية والتنقل
package com.agon.app

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.animation.AnimatedContentTransitionScope
import androidx.compose.animation.core.tween
import androidx.compose.animation.fadeIn
import androidx.compose.animation.fadeOut
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Home
import androidx.compose.material.icons.filled.Settings
import androidx.compose.material.icons.outlined.Home
import androidx.compose.material.icons.outlined.Settings
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.NavigationBar
import androidx.compose.material3.NavigationBarItem
import androidx.compose.material3.NavigationBarItemDefaults
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Brush
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.Dp
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.NavHostController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.navigation.compose.rememberNavController
import com.agon.app.data.Subscriber
import com.agon.app.ui.screens.AddEditSubscriberScreen
import com.agon.app.ui.screens.HomeScreen
import com.agon.app.ui.screens.SettingsScreen
import com.agon.app.ui.screens.backgroundGradients
import com.agon.app.ui.theme.AgonAppTheme
import com.agon.app.ui.theme.TealPrimary
import com.agon.app.viewmodel.MainViewModel

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        enableEdgeToEdge()
        super.onCreate(savedInstanceState)
        setContent {
            val viewModel: MainViewModel = viewModel()
            val darkMode by viewModel.darkMode.collectAsState()
            
            AgonAppTheme(darkTheme = darkMode) {
                MainApp(viewModel = viewModel)
            }
        }
    }
}

@Composable
fun MainApp(viewModel: MainViewModel) {
    val navController = rememberNavController()
    val backgroundIndex by viewModel.backgroundIndex.collectAsState()
    
    var selectedSubscriber by remember { mutableStateOf<Subscriber?>(null) }

    Box(modifier = Modifier.fillMaxSize()) {
        val currentGradient = backgroundGradients.getOrNull(backgroundIndex)
        if (currentGradient != null && currentGradient.colors.size > 1) {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .background(
                        Brush.linearGradient(
                            colors = currentGradient.colors.map { it.copy(alpha = 0.15f) },
                            start = androidx.compose.ui.geometry.Offset(0f, 0f),
                            end = androidx.compose.ui.geometry.Offset(
                                Float.POSITIVE_INFINITY,
                                Float.POSITIVE_INFINITY
                            )
                        )
                    )
            )
        }
        
        Scaffold(
            modifier = Modifier.fillMaxSize(),
            containerColor = Color.Transparent,
            bottomBar = { 
                val navBackStackEntry by navController.currentBackStackEntryAsState()
                val currentRoute = navBackStackEntry?.destination?.route
                
                if (currentRoute == "home" || currentRoute == "settings") {
                    BottomNav(navController) 
                }
            },
        ) { innerPadding ->
            NavHost(
                navController = navController,
                startDestination = "home",
                modifier = Modifier.padding(innerPadding),
                enterTransition = {
                    fadeIn(animationSpec = tween(300)) + slideIntoContainer(
                        AnimatedContentTransitionScope.SlideDirection.Start, tween(300))
                },
                exitTransition = {
                    fadeOut(animationSpec = tween(300)) + slideOutOfContainer(
                        AnimatedContentTransitionScope.SlideDirection.Start, tween(300))
                },
                popEnterTransition = {
                    fadeIn(animationSpec = tween(300)) + slideIntoContainer(
                        AnimatedContentTransitionScope.SlideDirection.End, tween(300))
                },
                popExitTransition = {
                    fadeOut(animationSpec = tween(300)) + slideOutOfContainer(
                        AnimatedContentTransitionScope.SlideDirection.End, tween(300))
                }
            ) {
                composable("home") { 
                    HomeScreen(
                        viewModel = viewModel,
                        onAddSubscriber = { selectedSubscriber = null; navController.navigate("add_subscriber") },
                        onEditSubscriber = { subscriber -> selectedSubscriber = subscriber; navController.navigate("edit_subscriber") },
                        backgroundIndex = backgroundIndex
                    )
                }
                composable("settings") { SettingsScreen(viewModel = viewModel) }
                composable("add_subscriber") { AddEditSubscriberScreen(viewModel, null) { navController.popBackStack() } }
                composable("edit_subscriber") { AddEditSubscriberScreen(viewModel, selectedSubscriber) { navController.popBackStack() } }
            }
        }
    }
}

@Composable
fun BottomNav(navController: NavHostController) {
    val navBackStackEntry by navController.currentBackStackEntryAsState()
    val currentRoute = navBackStackEntry?.destination?.route

    NavigationBar(containerColor = MaterialTheme.colorScheme.surface.copy(alpha = 0.95f), tonalElevation = Dp(8f)) {
        NavigationBarItem(
            icon = { Icon(if (currentRoute == "home") Icons.Filled.Home else Icons.Outlined.Home, "الرئيسية") },
            label = { Text("الرئيسية", fontWeight = if (currentRoute == "home") FontWeight.Bold else FontWeight.Normal) },
            selected = currentRoute == "home",
            onClick = { navController.navigate("home") { popUpTo("home") { inclusive = true } } },
            colors = NavigationBarItemDefaults.colors(
                selectedIconColor = TealPrimary, selectedTextColor = TealPrimary,
                indicatorColor = TealPrimary.copy(alpha = 0.1f)))
        NavigationBarItem(
            icon = { Icon(if (currentRoute == "settings") Icons.Filled.Settings else Icons.Outlined.Settings, "الإعدادات") },
            label = { Text("الإعدادات", fontWeight = if (currentRoute == "settings") FontWeight.Bold else FontWeight.Normal) },
            selected = currentRoute == "settings",
            onClick = { navController.navigate("settings") { popUpTo("home") } },
            colors = NavigationBarItemDefaults.colors(
                selectedIconColor = TealPrimary, selectedTextColor = TealPrimary,
                indicatorColor = TealPrimary.copy(alpha = 0.1f)))
    }
}
2️⃣ Subscriber.kt - نموذج بيانات المشترك
package com.agon.app.data
import kotlinx.serialization.Serializable

@Serializable
data class Subscriber(
    val id: String = java.util.UUID.randomUUID().toString(),
    val name: String,
    val phoneNumber: String = "",
    val subscriptionAmount: Double,
    val paidAmount: Double = 0.0,
    val subscriptionStartDate: Long = System.currentTimeMillis(),
    val subscriptionDays: Int = 30,
    val notes: String = "",
    val isActive: Boolean = true
) {
    val remainingDebt: Double get() = subscriptionAmount - paidAmount
    val isPaid: Boolean get() = paidAmount >= subscriptionAmount
    val subscriptionEndDate: Long get() = subscriptionStartDate + (subscriptionDays * 24 * 60 * 60 * 1000L)
    val daysRemaining: Int get() = ((subscriptionEndDate - System.currentTimeMillis()) / (24 * 60 * 60 * 1000L)).toInt()
    val isExpiringSoon: Boolean get() = daysRemaining in 0..1 && !isPaid
    val isExpired: Boolean get() = daysRemaining < 0
}
3️⃣ SubscriberRepository.kt - حفظ بيانات المشتركين
package com.agon.app.data
import android.content.Context
import androidx.datastore.preferences.core.*
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import kotlinx.serialization.encodeToString
import kotlinx.serialization.json.Json

val Context.dataStore by preferencesDataStore(name = "subscribers")

class SubscriberRepository(private val context: Context) {
    private val SUBSCRIBERS_KEY = stringPreferencesKey("subscribers_list")
    private val json = Json { ignoreUnknownKeys = true }

    val subscribers: Flow<List<Subscriber>> = context.dataStore.data.map { prefs ->
        try { json.decodeFromString(prefs[SUBSCRIBERS_KEY] ?: "[]") } catch(e: Exception) { emptyList() }
    }

    suspend fun addSubscriber(subscriber: Subscriber) {
        context.dataStore.edit { prefs ->
            val list = try { json.decodeFromString<List<Subscriber>>(prefs[SUBSCRIBERS_KEY] ?: "[]") } catch(e: Exception) { emptyList() }
            prefs[SUBSCRIBERS_KEY] = json.encodeToString(list + subscriber)
        }
    }

    suspend fun updateSubscriber(subscriber: Subscriber) {
        context.dataStore.edit { prefs ->
            val list = try { json.decodeFromString<List<Subscriber>>(prefs[SUBSCRIBERS_KEY] ?: "[]") } catch(e: Exception) { emptyList() }
            prefs[SUBSCRIBERS_KEY] = json.encodeToString(list.map { if(it.id == subscriber.id) subscriber else it })
        }
    }

    suspend fun deleteSubscriber(id: String) {
        context.dataStore.edit { prefs ->
            val list = try { json.decodeFromString<List<Subscriber>>(prefs[SUBSCRIBERS_KEY] ?: "[]") } catch(e: Exception) { emptyList() }
            prefs[SUBSCRIBERS_KEY] = json.encodeToString(list.filter { it.id != id })
        }
    }

    suspend fun renewSubscription(id: String) {
        context.dataStore.edit { prefs ->
            val list = try { json.decodeFromString<List<Subscriber>>(prefs[SUBSCRIBERS_KEY] ?: "[]") } catch(e: Exception) { emptyList() }
            prefs[SUBSCRIBERS_KEY] = json.encodeToString(list.map {
                if(it.id == id) it.copy(subscriptionStartDate = System.currentTimeMillis(), paidAmount = 0.0) else it })
        }
    }
}
4️⃣ SettingsRepository.kt - حفظ الإعدادات
package com.agon.app.data
import android.content.Context
import androidx.datastore.preferences.core.*
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

class SettingsRepository(private val context: Context) {
    val darkMode: Flow<Boolean> = context.dataStore.data.map { it[booleanPreferencesKey("dark_mode")] ?: false }
    val notificationDays: Flow<Int> = context.dataStore.data.map { it[intPreferencesKey("notification_days")] ?: 1 }
    val currency: Flow<String> = context.dataStore.data.map { it[stringPreferencesKey("currency")] ?: "IQD" }
    val backgroundIndex: Flow<Int> = context.dataStore.data.map { it[intPreferencesKey("background_index")] ?: 0 }
    val notificationsEnabled: Flow<Boolean> = context.dataStore.data.map { it[booleanPreferencesKey("notifications_enabled")] ?: true }
    val defaultSubscriptionDays: Flow<Int> = context.dataStore.data.map { it[intPreferencesKey("default_subscription_days")] ?: 30 }

    suspend fun setDarkMode(v: Boolean) = context.dataStore.edit { it[booleanPreferencesKey("dark_mode")] = v }
    suspend fun setNotificationDays(d: Int) = context.dataStore.edit { it[intPreferencesKey("notification_days")] = d }
    suspend fun setCurrency(c: String) = context.dataStore.edit { it[stringPreferencesKey("currency")] = c }
    suspend fun setBackgroundIndex(i: Int) = context.dataStore.edit { it[intPreferencesKey("background_index")] = i }
    suspend fun setNotificationsEnabled(e: Boolean) = context.dataStore.edit { it[booleanPreferencesKey("notifications_enabled")] = e }
    suspend fun setDefaultSubscriptionDays(d: Int) = context.dataStore.edit { it[intPreferencesKey("default_subscription_days")] = d }
}
5️⃣ MainViewModel.kt - المنطق البرمجي
package com.agon.app.viewmodel
import android.app.Application
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import com.agon.app.data.*
import kotlinx.coroutines.flow.*

class MainViewModel(app: Application) : AndroidViewModel(app) {
    private val subRepo = SubscriberRepository(app)
    private val settingsRepo = SettingsRepository(app)

    val subscribers: StateFlow<List<Subscriber>> = subRepo.subscribers.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), emptyList())
    val darkMode: StateFlow<Boolean> = settingsRepo.darkMode.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), false)
    val notificationDays: StateFlow<Int> = settingsRepo.notificationDays.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), 1)
    val currency: StateFlow<String> = settingsRepo.currency.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), "IQD")
    val backgroundIndex: StateFlow<Int> = settingsRepo.backgroundIndex.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), 0)
    val notificationsEnabled: StateFlow<Boolean> = settingsRepo.notificationsEnabled.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), true)
    val defaultSubscriptionDays: StateFlow<Int> = settingsRepo.defaultSubscriptionDays.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), 30)
    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery.asStateFlow()

    fun setSearchQuery(q: String) { _searchQuery.value = q }
    fun addSubscriber(s: Subscriber) { viewModelScope.launch { subRepo.addSubscriber(s) } }
    fun updateSubscriber(s: Subscriber) { viewModelScope.launch { subRepo.updateSubscriber(s) } }
    fun deleteSubscriber(id: String) { viewModelScope.launch { subRepo.deleteSubscriber(id) } }
    fun renewSubscription(id: String) { viewModelScope.launch { subRepo.renewSubscription(id) } }
    fun setDarkMode(v: Boolean) { viewModelScope.launch { settingsRepo.setDarkMode(v) } }
    fun setNotificationDays(d: Int) { viewModelScope.launch { settingsRepo.setNotificationDays(d) } }
    fun setCurrency(c: String) { viewModelScope.launch { settingsRepo.setCurrency(c) } }
    fun setBackgroundIndex(i: Int) { viewModelScope.launch { settingsRepo.setBackgroundIndex(i) } }
    fun setNotificationsEnabled(e: Boolean) { viewModelScope.launch { settingsRepo.setNotificationsEnabled(e) } }
    fun setDefaultSubscriptionDays(d: Int) { viewModelScope.launch { settingsRepo.setDefaultSubscriptionDays(d) } }

    val totalDebt: Double get() = subscribers.value.sumOf { it.remainingDebt }
    val totalPaid: Double get() = subscribers.value.sumOf { it.paidAmount }
    val activeSubscribersCount: Int get() = subscribers.value.count { it.isActive && !it.isExpired }
    val expiredSubscribersCount: Int get() = subscribers.value.count { it.isExpired }
}
6️⃣ Color.kt - الألوان
package com.agon.app.ui.theme
import androidx.compose.ui.graphics.Color

val TealPrimary = Color(0xFF00897B); val TealDark = Color(0xFF00695C); val TealLight = Color(0xFF4DB6AC)
val GoldAccent = Color(0xFFFFB300); val GoldDark = Color(0xFFFFA000)
val SurfaceLight = Color(0xFFF5F5F5); val BackgroundLight = Color(0xFFFFFFFF); val OnSurfaceLight = Color(0xFF212121)
val ErrorRed = Color(0xFFE53935); val SuccessGreen = Color(0xFF43A047); val WarningOrange = Color(0xFFFF9800)
val TealPrimaryDark = Color(0xFF4DB6AC); val TealSecondaryDark = Color(0xFF80CBC4); val GoldAccentDark = Color(0xFFFFCA28)
val SurfaceDark = Color(0xFF1E1E1E); val BackgroundDark = Color(0xFF121212); val OnSurfaceDark = Color(0xFFE0E0E0)
val CardGreen = Color(0xFFE8F5E9); val CardYellow = Color(0xFFFFF8E1); val CardRed = Color(0xFFFFEBEE); val CardBlue = Color(0xFFE3F2FD)
7️⃣ Theme.kt - السمة
package com.agon.app.ui.theme
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.graphics.Color

private val DarkColorScheme = darkColorScheme(primary=TealPrimaryDark, onPrimary=Color.Black, secondary=TealSecondaryDark, onSecondary=Color.Black, tertiary=GoldAccentDark, onTertiary=Color.Black, background=BackgroundDark, onBackground=OnSurfaceDark, surface=SurfaceDark, onSurface=OnSurfaceDark, surfaceVariant=Color(0xFF2D2D2D), onSurfaceVariant=Color(0xFFBDBDBD), error=ErrorRed, onError=Color.White)
private val LightColorScheme = lightColorScheme(primary=TealPrimary, onPrimary=Color.White, secondary=TealLight, onSecondary=Color.Black, tertiary=GoldAccent, onTertiary=Color.Black, background=BackgroundLight, onBackground=OnSurfaceLight, surface=SurfaceLight, onSurface=OnSurfaceLight, surfaceVariant=Color(0xFFE0E0E0), onSurfaceVariant=Color(0xFF616161), error=ErrorRed, onError=Color.White)

@Composable
fun AgonAppTheme(darkTheme: Boolean = isSystemInDarkTheme(), content: @Composable () -> Unit) {
    MaterialTheme(colorScheme = if(darkTheme) DarkColorScheme else LightColorScheme, content = content)
}
8️⃣ HomeScreen.kt - قائمة المشتركين (كامل - عرضت أعلاه)
9️⃣ AddEditSubscriberScreen.kt - إضافة/تعديل مشترك (كامل - عرضت أعلاه)
🔟 SettingsScreen.kt - الإعدادات + تدرجات الألوان (كامل - عرضت أعلاه)
