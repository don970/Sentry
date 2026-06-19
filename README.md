<p align="center">
  <img src="./sentry_concept_logo.png" alt="Sentry Logo" width="220" />
</p>

<h1 align="center">🛡️ Sentry OSINT</h1>

<p align="center">
  <strong>A High-Performance Android Reconnaissance, Identity Intelligence, & Local Network Scanning Suite</strong>
</p>

<p align="center">
  <a href="https://kotlinlang.org/"><img src="https://img.shields.io/badge/Language-Kotlin-purple?style=for-the-badge&logo=kotlin" alt="Kotlin" /></a>
  <a href="https://developer.android.com/"><img src="https://img.shields.io/badge/Platform-Android-3DDC84?style=for-the-badge&logo=android&logoColor=white" alt="Android" /></a>
  <a href="https://developer.android.com/jetpack/compose"><img src="https://img.shields.io/badge/UI-Jetpack%20Compose-4285F4?style=for-the-badge&logo=jetpackcompose&logoColor=white" alt="Jetpack Compose" /></a>
  <br />
  <a href="https://ktor.io/"><img src="https://img.shields.io/badge/Network-Ktor%20HTTP-008080?style=for-the-badge&logo=ktor&logoColor=white" alt="Ktor" /></a>
  <a href="https://developer.android.com/training/data-storage/room"><img src="https://img.shields.io/badge/Database-Room%20%2F%20SQLite-E34F26?style=for-the-badge&logo=sqlite&logoColor=white" alt="Room" /></a>
  <a href="https://dagger.dev/hilt/"><img src="https://img.shields.io/badge/DI-Dagger%20Hilt-7F3FBF?style=for-the-badge" alt="Dagger Hilt" /></a>
</p>

---

## 📖 Overview

**Sentry** is a powerful, mobile-first Open-Source Intelligence (OSINT) and network reconnaissance application built natively for Android. Tailored for cybersecurity auditors, OSINT practitioners, and systems administrators, Sentry bridges the gap between desktop intelligence-gathering utilities and the portability of mobile devices. 

Conduct complete digital footprint analysis, track online handles across dozens of networks, audit local subnets, parse live server HTTP configurations, and cross-reference identities—all directly from a sleek, fast, and secure local interface.

---

## 🚀 Key Modules & Capabilities

Sentry is engineered around high-performance, asynchronous workflows. Each component runs independently and publishes structured updates directly to your screen.

```
                  ┌─────────────────────────────────────────┐
                  │              Sentry OSINT               │
                  └─────────────────────────────────────────┘
                                       │
       ┌───────────────────┬───────────┴───────────┬───────────────────┐
       ▼                   ▼                       ▼                   ▼
┌─────────────┐     ┌─────────────┐         ┌─────────────┐     ┌─────────────┐
│   LAN SCAN  │     │  IDENTITY   │         │ BACKGROUND  │     │ DEEP SEARCH │
│  & Recon    │     │ Correlation │         │  Profiler   │     │  Dork/Web   │
└─────────────┘     └─────────────┘         └─────────────┘     └─────────────┘
```

### 📶 1. Host & Local Network Scanner
*   **Subnet Discovery:** Automatically detects your local IP network structure and performs non-intrusive `/24` subnet sweeps to discover active network devices.
*   **Targeted Port Scanner:** Scans high-priority service ports (`80`, `443`, `22`, `135`, `445`, `8080`) to discover open services, web interfaces, and diagnostic endpoints on active hosts.
*   **Dynamic DNS Resolution:** Resolves active network hostnames concurrently using system DNS tables (`InetAddress`).

### 👤 2. Identity & Social Handles Correlation
*   **50+ Platform Verification:** Cross-references search parameters against 50+ mainstream and niche websites spanning Social, Professional, Dating, Developer, Messaging/AI, and Adult categories.
*   **Intelligent Handle Derivation:** Automatically extrapolates possible internet handle variants (e.g., `firstlast`, `first_last`, `first.last`, `flast`) from basic target names to maximize lookup surface.
*   **Parallel Non-Blocking Checks:** Executes lightweight asynchronous HTTP lookups concurrently utilizing `Ktor`'s non-blocking coroutine engine, delivering complete platform reports in seconds.

### 🕵️ 3. Advanced Subject Profiling (Background Check)
*   **Structured Profiles:** Builds local directories of investigative targets, incorporating first names, last names, physical locations, emails, aliases, and known relationships.
*   **Affiliate Network Tree:** Maps familial structures, companions, and colleagues into a unified profile.
*   **Visual Reverse Search:** Attaches visual reference photos and initiates deep image queries directly using Android system intents to launch Google Lens or Meta AI for image-based reverse search tracking.

### 🔍 4. Google Dorking & Public Mentions
*   **Google Dork Engine:** Custom client to perform query dorking and scrape search results safely.
*   **Mobile-Optimized Scraper:** Bypasses basic mobile layout constraints, isolating title elements, actual destinations, and description snippets using advanced regex matches.

### 🛡️ 5. Google Account Verification
*   **Active Presence Checks:** Leverages standard Google account recovery/RTT endpoints (`mail.google.com/mail/gxlu`) to check if a specific email address is registered as an active Google account, returning quick, accurate results.

### 📁 6. Historic Internet Archive (Wayback Machine) Audits
*   **Availability Tracking:** Direct integration with the Internet Archive's Wayback availability API (`archive.org/wayback`) to quickly check if a target domain or site has historically saved snapshots, providing direct URL paths to snapshots.

### 🔬 7. HTTP Header & Security Analyzer
*   **Metadata Extractor:** Inspects raw HTTP headers of target servers, exposing server platforms, framework configurations, security policies, cache rules, and cookie fields.

---

## 🛠️ Technology Stack & Architecture

Sentry is built strictly adhering to modern Android development standards, ensuring robustness, modularity, and high-speed execution.

*   **Architecture Pattern:** Clean Architecture with Model-View-ViewModel (MVVM) principles.
*   **UI Framework:** Jetpack Compose (Material 3) for reactive, modern layouts, consistent spacing, and smooth transitions.
*   **Asynchronous Engine:** Kotlin Coroutines & Asynchronous Flows for concurrent background tasks (e.g., executing 50+ HTTP requests simultaneously).
*   **HTTP Engine:** Ktor Client with OkHttp backend, utilizing customized User-Agents, connection pooling, and request-level configurations.
*   **Local Persistence:** Room ORM wrapping SQLite for saving target profile sheets, search histories, and network logs locally.
*   **Dependency Injection:** Dagger Hilt for clean dependency graph construction and lifecycle management.

---

## 🔬 Core Code Showcase

Here are some highlights of Sentry's highly optimized, asynchronous code design:

### Parallel Social Handle Checker (Identity Engine)
This method maps and queries dozens of sites concurrently, ensuring the interface remains snappy and users get results instantly:

```kotlin
suspend fun checkUsername(username: String): List<IdentityResult> = withContext(Dispatchers.IO) {
    if (username.contains(" ")) return@withContext emptyList()
    
    Log.d(TAG, "Checking username: $username across ${sites.size} sites")
    sites.map { (name, data) ->
        async {
            val (baseUrl, category) = data
            val url = "$baseUrl$username"
            try {
                val response: HttpResponse = client.get(url) {
                    header("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...")
                }
                val exists = response.status.value == 200
                IdentityResult(name, url, exists, category)
            } catch (e: Exception) {
                IdentityResult(name, url, false, category)
            }
        }
    }.awaitAll()
}
```

### Google Account Verification Pattern
This smart check validates Google email presence by identifying cookies returned by GMail endpoints:

```kotlin
suspend fun verifyGoogleAccount(email: String): Boolean = withContext(Dispatchers.IO) {
    try {
        val url = "https://mail.google.com/mail/gxlu?email=$email"
        val response: HttpResponse = client.get(url)
        val hasCookie = response.headers.entries().any { it.key.lowercase() == "set-cookie" }
        hasCookie
    } catch (e: Exception) {
        false
    }
}
```

---

## 🚀 Building & Installing

### Prerequisites
*   **Android Studio Jellyfish / Ladybug (2024.1+)** or command line tools with Android SDK 34.
*   **Java Development Kit (JDK):** JDK 17 (or compatible newer version).
*   **Gradle Build System:** Handles Kotlin DSL and dependencies automatically.

### Command Line Build
To compile Sentry and build an APK via terminal:

1.  **Clone the Repository** and navigate to the root directory:
    ```bash
    git clone https://github.com/supersploit/sentry.git
    cd sentry
    ```
2.  **Generate Debug APK:**
    ```bash
    ./gradlew assembleDebug
    ```
    Once completed, find your APK at:
    `app/build/outputs/apk/debug/app-debug.apk`

3.  **Run Tests:**
    ```bash
    ./gradlew test
    ```

---

## ⚠️ Legal Disclaimer

This application is designed strictly for **legitimate security auditing, open-source intelligence research, and academic analysis**. The developers assume no liability for the misuse of this tool for malicious, illegal, or unauthorized surveillance or data-harvesting purposes. Users are solely responsible for ensuring compliance with local laws, regulations, and target platforms' terms of service. Always obtain explicit permission before scanning network devices or auditing infrastructure.
