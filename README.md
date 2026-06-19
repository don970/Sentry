## 📖 Overview

**Sentry** is a powerful, mobile-first Open-Source Intelligence (OSINT) and network reconnaissance application built natively for Android. Designed specifically for cybersecurity auditors, OSINT practitioners, and systems administrators, Sentry bridges the gap between desktop intelligence-gathering utilities and the tactical portability of mobile devices.

Conduct complete digital footprint analysis, track online handles across dozens of networks, audit local subnets, parse live server HTTP configurations, and cross-reference identities—all from a sleek, fast, and secure local interface.

---

## 🚀 Architecture & Workflow

Sentry is engineered around high-performance, asynchronous workflows. Each module runs independently, publishing structured data directly to the user interface without blocking the main thread.

```text
                  ┌─────────────────────────────────────────┐
                  │              Sentry OSINT               │
                  └─────────────────────────────────────────┘
                                       │
       ┌───────────────────┬───────────┴───────────┬───────────────────┐
       ▼                   ▼                       ▼                   ▼
┌─────────────┐     ┌─────────────┐         ┌─────────────┐     ┌─────────────┐
│  LAN SCAN   │     │  IDENTITY   │         │ BACKGROUND  │     │ DEEP SEARCH │
│  & Recon    │     │ Correlation │         │  Profiler   │     │  Dork/Web   │
└─────────────┘     └─────────────┘         └─────────────┘     └─────────────┘

```

---

## 🔍 Core Capabilities

### 📶 Host & Local Network Scanner

* **Subnet Discovery:** Automatically detects local IP network architecture and executes non-intrusive `/24` subnet sweeps to map active devices.
* **Targeted Port Scanner:** Scans high-priority service ports (`80`, `443`, `22`, `135`, `445`, `8080`) to uncover web interfaces, open services, and diagnostic endpoints.
* **Dynamic DNS Resolution:** Resolves active network hostnames concurrently using system DNS tables.

### 👤 Identity & Social Correlation

* **50+ Platform Verification:** Cross-references handles against mainstream and niche platforms (Social, Professional, Developer, Messaging, etc.).
* **Intelligent Handle Derivation:** Extrapolates likely username variants (e.g., `firstlast`, `first_last`, `first.last`) to maximize your search surface.
* **Parallel Execution:** Executes lightweight HTTP lookups concurrently utilizing Ktor's non-blocking coroutine engine, delivering comprehensive reports in seconds.

### 🕵️ Advanced Subject Profiling

* **Structured Local Profiles:** Build robust offline directories of targets, incorporating names, physical locations, emails, aliases, and known affiliations.
* **Visual Reverse Search:** Attach reference photos and seamlessly hand off deep image queries to Google Lens or Meta AI via Android system intents.

### 🌐 Web & Infrastructure Recon

* **Mobile Dorking Engine:** A custom client designed to execute Google Dorks and safely extract search results, bypassing mobile layout constraints via regex-driven DOM parsing.
* **Historic Internet Archive Audits:** Direct integration with the Wayback Machine API to verify if a target domain has historic snapshots, providing instant access URLs.
* **HTTP Header Analyzer:** Inspects raw HTTP headers to expose server platforms, security policies, and cache rules.
* **Google Account Verification:** Leverages Google recovery endpoints to silently verify the existence of specific Gmail accounts.

---

## 🛠️ Technology Stack

Sentry strictly adheres to modern Android development standards, prioritizing robustness, modularity, and high-speed execution.

| Layer | Technology | Description |
| --- | --- | --- |
| **Architecture** | Clean Arch / MVVM | Enforces separation of concerns and testability. |
| **UI Framework** | Jetpack Compose (M3) | Reactive, state-driven UI with modern Material 3 design. |
| **Concurrency** | Kotlin Coroutines & Flows | Asynchronous engine for handling 50+ simultaneous network tasks. |
| **Networking** | Ktor Client & OkHttp | Configurable HTTP engine with custom User-Agents and connection pooling. |
| **Persistence** | Room ORM (SQLite) | Secure, local-only database for profiles, history, and scan logs. |
| **Dependency Injection** | Dagger Hilt | Streamlined dependency graph and lifecycle management. |

---

## 🔬 Code Showcase

Sentry's backend is highly optimized. Below are examples of how i handle rapid, concurrent OSINT tasks.

### Parallel Identity Engine

Maps and queries dozens of endpoints simultaneously, ensuring immediate feedback on the UI layer:

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

### Silent Account Verification

Validates Google email presence by evaluating cookie headers returned by GMail endpoints:

```kotlin
suspend fun verifyGoogleAccount(email: String): Boolean = withContext(Dispatchers.IO) {
    try {
        val url = "https://mail.google.com/mail/gxlu?email=$email"
        val response: HttpResponse = client.get(url)
        // A valid account triggers a Set-Cookie header response
        response.headers.entries().any { it.key.lowercase() == "set-cookie" }
    } catch (e: Exception) {
        false
    }
}

```

---

## 🗺️ Roadmap (Coming Soon)

* [ ] **Alpha Release:** Core APK distribution for early testers.
* [ ] **Custom Dork Library:** Save and load your own regex-based query lists.
* [ ] **Dark Web Market Search:** Integration with Tor proxy routing.
* [ ] **Export/Reporting:** Generate PDF or JSON reports for clients and audits.

---

## ⚙️ Building & Installing

### Prerequisites

* **Android Studio Jellyfish / Ladybug (2024.1+)**
* **Java Development Kit (JDK 17+)**

### Command Line Build

To compile Sentry and build an APK locally:

1. **Clone the Repository:**
```bash
git clone https://github.com/supersploit/sentry.git
cd sentry

```


2. **Generate Debug APK:**
```bash
./gradlew assembleDebug

```


> **Note:** Upon successful compilation, your APK will be located at `app/build/outputs/apk/debug/app-debug.apk`.


3. **Run Unit Tests:**
```bash
./gradlew test

```



---

## ⚠️ Legal Disclaimer

This application is designed strictly for **legitimate security auditing, authorized open-source intelligence research, and academic analysis**.

The developers and contributors assume **no liability** for the misuse of this tool for malicious, illegal, or unauthorized surveillance or data-harvesting purposes. Users are solely responsible for ensuring complete compliance with local laws, international regulations, and target platforms' terms of service. **Always obtain explicit, written permission before scanning network devices or auditing infrastructure.**
