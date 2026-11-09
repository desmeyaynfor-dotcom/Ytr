package com.tuservicio.cazaanuncios

import android.accessibilityservice.AccessibilityService
import android.accessibilityservice.AccessibilityServiceInfo
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.provider.Settings
import android.util.Log
import android.view.accessibility.AccessibilityEvent
import android.widget.Toast
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

// ==========================================
// 1. ACTIVIDAD PRINCIPAL Y UI (Jetpack Compose)
// ==========================================
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            MaterialTheme {
                MainScreen(
                    onEnableAccessibility = { openAccessibilitySettings() },
                    onUninstallApp = { pkg -> uninstallApp(pkg) }
                )
            }
        }
    }

    private fun openAccessibilitySettings() {
        startActivity(Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS))
    }

    private fun uninstallApp(packageName: String) {
        val intent = Intent(Intent.ACTION_DELETE).apply {
            data = Uri.parse("package:$packageName")
        }
        startActivity(intent)
    }
}

@Composable
fun MainScreen(onEnableAccessibility: () -> Unit, onUninstallApp: (String) -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "🛡️ Adware & Malware Detector",
            fontSize = 22.sp,
            style = MaterialTheme.typography.headlineMedium
        )

        Spacer(modifier = Modifier.height(16.dp))

        Card(
            modifier = Modifier.fillMaxWidth(),
            colors = CardDefaults.cardColors(containerColor = Color(0xFF1E293B))
        ) {
            Column(modifier = Modifier.padding(16.dp)) {
                Text("Escudo en Segundo Plano", color = Color.White, fontSize = 18.sp)
                Text(
                    "Activa el servicio para detectar qué app abre anuncios emergentes.",
                    color = Color.LightGray,
                    fontSize = 14.sp
                )
                Spacer(modifier = Modifier.height(8.dp))
                Button(onClick = onEnableAccessibility) {
                    Text("Activar Escudo Caza-Anuncios")
                }
            }
        }

        Spacer(modifier = Modifier.height(20.dp))

        Text(
            text = "Acciones Rápidas de Limpieza",
            fontSize = 18.sp,
            modifier = Modifier.align(Alignment.Start)
        )

        Spacer(modifier = Modifier.height(10.dp))

        LazyColumn {
            item {
                AppItemCard(
                    appName = "App Sospechosa Ejemplo",
                    packageName = "com.ejemplo.adware",
                    reason = "Permiso de superposición activado",
                    onUninstall = { onUninstallApp("com.ejemplo.adware") }
                )
            }
        }
    }
}

@Composable
fun AppItemCard(appName: String, packageName: String, reason: String, onUninstall: () -> Unit) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(vertical = 4.dp),
        colors = CardDefaults.cardColors(containerColor = Color(0xFFFEE2E2))
    ) {
        Row(
            modifier = Modifier
                .padding(12.dp)
                .fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Column(modifier = Modifier.weight(1f)) {
                Text(appName, color = Color(0xFF991B1B), fontSize = 16.sp)
                Text(packageName, color = Color.DarkGray, fontSize = 12.sp)
                Text("⚠️ $reason", color = Color(0xFFB91C1C), fontSize = 12.sp)
            }
            Button(
                onClick = onUninstall,
                colors = ButtonDefaults.buttonColors(containerColor = Color(0xFFDC2626))
            ) {
                Text("Borrar", color = Color.White)
            }
        }
    }
}

// ==========================================
// 2. SERVICIO DE DETECCIÓN DE ANUNCIOS EN SEGUNDO PLANO
// ==========================================
class AdDetectorService : AccessibilityService() {

    override fun onAccessibilityEvent(event: AccessibilityEvent?) {
        if (event == null) return

        if (event.eventType == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED) {
            val packageName = event.packageName?.toString() ?: return
            val className = event.className?.toString() ?: ""

            if (packageName != applicationContext.packageName && !packageName.startsWith("com.android")) {
                if (className.contains("ad", ignoreCase = true) || 
                    className.contains("interstitial", ignoreCase = true) || 
                    className.contains("mobfox", ignoreCase = true) ||
                    className.contains("unity3d.ads", ignoreCase = true)) {

                    Log.w("AdDetector", "¡Anuncio invasivo detectado desde: $packageName!")
                    
                    Toast.makeText(
                        this,
                        "⚠️ Anuncio provocado por: $packageName. ¡Toca para desinstalar!",
                        Toast.LENGTH_LONG
                    ).show()
                }
            }
        }
    }

    override fun onInterrupt() {
        Log.e("AdDetector", "El servicio de accesibilidad fue interrumpido.")
    }

    override fun onServiceConnected() {
        super.onServiceConnected()
        val info = AccessibilityServiceInfo().apply {
            eventTypes = AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED
            feedbackType = AccessibilityServiceInfo.FEEDBACK_GENERIC
            flags = AccessibilityServiceInfo.FLAG_INCLUDE_NOT_IMPORTANT_VIEWS
        }
        this.serviceInfo = info
    }
}

// ==========================================
// 3. ANALIZADOR DE PERMISOS DE APLICACIONES
// ==========================================
data class AppReport(
    val appName: String,
    val packageName: String,
    val isDangerous: Boolean,
    val suspiciousPermissions: List<String>
)

class ApkAnalyzer(private val context: Context) {

    private val dangerousPermissionsList = listOf(
        "android.permission.SYSTEM_ALERT_WINDOW",
        "android.permission.RECEIVE_BOOT_COMPLETED",
        "android.permission.REQUEST_INSTALL_PACKAGES",
        "android.permission.BIND_ACCESSIBILITY_SERVICE",
        "android.permission.PACKAGE_USAGE_STATS"
    )

    fun analyzeInstalledApp(packageName: String): AppReport {
        val pm = context.packageManager
        var appName = packageName
        val foundSuspicious = mutableListOf<String>()

        try {
            val appInfo = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                pm.getApplicationInfo(packageName, PackageManager.ApplicationInfoFlags.of(0))
            } else {
                @Suppress("DEPRECATION")
                pm.getApplicationInfo(packageName, 0)
            }
            appName = pm.getApplicationLabel(appInfo).toString()

            val packageInfo = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                pm.getPackageInfo(packageName, PackageManager.PackageInfoFlags.of(PackageManager.GET_PERMISSIONS.toLong()))
            } else {
                @Suppress("DEPRECATION")
                pm.getPackageInfo(packageName, PackageManager.GET_PERMISSIONS)
            }

            packageInfo.requestedPermissions?.forEach { perm ->
                if (dangerousPermissionsList.contains(perm)) {
                    foundSuspicious.add(perm.replace("android.permission.", ""))
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }

        return AppReport(
            appName = appName,
            packageName = packageName,
            isDangerous = foundSuspicious.isNotEmpty(),
            suspiciousPermissions = foundSuspicious
        )
    }
}
