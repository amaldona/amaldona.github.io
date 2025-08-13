### **Veryfi Lens for Mobile — Cookbook for Cursor AI / Claude Code Integration**

#### **Overview & Purpose**

This guide explains how to integrate the **Veryfi Lens for Mobile Android SDK** into your Android app, particularly when working with LLMs like Cursor AI or Claude Code.

---

#### **Prerequisites**

* API and Maven credentials ([docs](https://docs.veryfi.com/lens/mobile/introduction/authentication/))
* Kotlin or Java project using **Gradle 8+** and **Java 17+**
* Minimum Android version: **6.0 (API Level 23)**

---

### **1. Obtain Credentials**

You need **two sets** of credentials:

1. **Maven credentials** — for downloading the SDK
   *(Maven username and password)*
2. **API credentials** — for accessing the Veryfi API
   *(Client ID, API key, username, URL)*

Steps:

1. If you don’t have a Veryfi account, [register here](https://app.veryfi.com/signup/api/).

2. In the Veryfi hub, go to **Settings → Keys**:

   * **API Auth Credentials** — copy your API credentials.
   * **Lens: Maven (Android)** — create your Maven credentials.

3. Add Maven credentials to your system environment (`~/.zshrc` or equivalent):

   ```bash
   export MAVEN_VERYFI_USERNAME=[USERNAME]
   export MAVEN_VERYFI_PASSWORD=[PASSWORD]
   ```

4. Verify credentials:

   ```bash
   curl -sS --head https://$MAVEN_VERYFI_USERNAME:$MAVEN_VERYFI_PASSWORD@nexus.veryfi.com/repository/maven-public/com/veryfi/lens/veryfi-lens-sdk/2.1.0.25/veryfi-lens-sdk-2.1.0.25.pom | grep "HTTP/2"
   ```

   * **Valid:** `HTTP/2 200`
   * **Invalid:** `HTTP/2 401` → recheck variables or regenerate credentials.

   To print stored variables:

   ```bash
   echo $MAVEN_VERYFI_USERNAME
   echo $MAVEN_VERYFI_PASSWORD
   ```

---

### **2. Install the SDK**

You’ll modify three files:
`settings.gradle` (or `.kts`), `app/build.gradle` (or `.kts`), and `AndroidManifest.xml`.

#### **a. Add Maven repository https://nexus.veryfi.com/repository/maven-releases/ using the system environment variables `MAVEN_VERYFI_USERNAME` and `MAVEN_VERYFI_PASSWORD` for authentication**

**Groovy (`settings.gradle`):**

```groovy
dependencyResolutionManagement {
    repositories {
        maven {
            url "https://nexus.veryfi.com/repository/maven-releases/"
            credentials {
                username = System.getenv("MAVEN_VERYFI_USERNAME")
                password = System.getenv("MAVEN_VERYFI_PASSWORD")
            }
            authentication {
                basic(BasicAuthentication)
            }
        }
    }
}
```

**Kotlin (`settings.gradle.kts`):**

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        maven {
            url = uri("https://nexus.veryfi.com/repository/maven-releases/")
            credentials {
                username = System.getenv("MAVEN_VERYFI_USERNAME")
                password = System.getenv("MAVEN_VERYFI_PASSWORD")
            }
            authentication {
                create<BasicAuthentication>("basic")
            }
        }
    }
}
```

#### **b. Prevent resource compression**

**Groovy (`app/build.gradle`):**

```groovy
android {
    androidResources {
        noCompress += 'veryfi'
    }
}
```

**Kotlin (`app/build.gradle.kts`):**

```kotlin
android {
    androidResources {
        noCompress += "veryfi"
    }
}
```

#### **c. Add dependency**

Replace `VERYFI_SDK_VERSION` with the [latest release version number](https://github.com/veryfi/veryfi-lens-receipts-android-demo/releases).

**Groovy:**

```groovy
dependencies {
    implementation "com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION"
}
```

**Kotlin:**

```kotlin
dependencies {
    implementation("com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION")
}
```

#### **d. Update AndroidManifest**

If using `android:allowBackup` or `android:usesCleartextTraffic`, add the `tools` namespace and merge rules:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools">
    <application
        tools:ignore="AllowBackup,GoogleAppIndexingWarning"
        tools:replace="android:allowBackup, android:usesCleartextTraffic">
    </application>
</manifest>
```

#### **e. Sync Project**

In Android Studio: **File → Sync Project with Gradle Files**.

---

### **3. Initialize the SDK**

**a. Import classes**

```kotlin
import com.veryfi.lens.VeryfiLens
import com.veryfi.lens.VeryfiLensDelegate
import com.veryfi.lens.helpers.DocumentType
import com.veryfi.lens.helpers.VeryfiLensCredentials
import com.veryfi.lens.helpers.VeryfiLensSettings
```

**b. Set credentials**

```kotlin
val credentials = VeryfiLensCredentials().apply {
    clientId = "CLIENT_ID"
    username = "USERNAME"
    apiKey = "API_KEY"
    url = "URL"
}
```

**c. Configure settings** ([Docs](https://docs.veryfi.com/lens/mobile/settings/))

```kotlin
val settings = VeryfiLensSettings().apply {
    autoRotateIsOn = true
    autoSubmitDocumentOnCapture = true
    documentTypes = arrayListOf(DocumentType.RECEIPT)
    galleryIsOn = false
    moreMenuIsOn = false
}
```

**d. Implement delegate**

```kotlin
class MyActivity : VeryfiLensDelegate {
    override fun veryfiLensClose(json: JSONObject) { /* handle */ }
    override fun veryfiLensError(json: JSONObject) { /* handle */ }
    override fun veryfiLensSuccess(json: JSONObject) { /* handle */ }
    override fun veryfiLensUpdate(json: JSONObject) { /* handle */ }
}
```

**e. Register and configure**

```kotlin
VeryfiLens.setDelegate(this)
VeryfiLens.configure(application, credentials, settings) { }
```

---

### **4. Start the Lens Camera**

```kotlin
VeryfiLens.showCamera()
```

Handle results in:

```kotlin
override fun veryfiLensSuccess(json: JSONObject) { /* handle */ }
```

---

### **5. Full Example — MyActivity.kt**

```kotlin
class MyActivity : AppCompatActivity(), VeryfiLensDelegate {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val credentials = VeryfiLensCredentials().apply {
            clientId = "CLIENT_ID"
            username = "USERNAME"
            apiKey = "API_KEY"
            url = "URL"
        }

        val settings = VeryfiLensSettings().apply {
            autoCaptureIsOn = true
            autoRotateIsOn = true
            documentTypes = arrayListOf(DocumentType.RECEIPT)
        }

        VeryfiLens.setDelegate(this)
        VeryfiLens.configure(application, credentials, settings) { }
    }

    override fun veryfiLensClose(json: JSONObject) { Log.d("Lens", json.toString(2)) }
    override fun veryfiLensError(json: JSONObject) { Log.d("Lens", json.toString(2)) }
    override fun veryfiLensSuccess(json: JSONObject) { Log.d("Lens", json.toString(2)) }
    override fun veryfiLensUpdate(json: JSONObject) { Log.d("Lens", json.toString(2)) }

    private fun onSomeButtonClicked() {
        VeryfiLens.showCamera()
    }
}
```

