### **Veryfi Lens for Mobile — Cookbook for Cursor AI / Claude Code Integration**

#### **Overview & Purpose**

This cookbook explains how to integrate the **Veryfi Lens for Mobile Android SDK** into Android apps—especially when using LLMs like Cursor AI or Claude Code.

#### **Prerequisites**

* API and Maven credentials ([Authentication guide](https://docs.veryfi.com/lens/mobile/introduction/authentication/))
* Kotlin or Java project with **Gradle 8** and **Java 17** or newer
* Minimum Android SDK **6.0 (API Level 23)**

---

#### **1. Obtaining Credentials**

You will need two sets of credentials:

1. **Maven credentials** — to download the SDK (username and password)
2. **API credentials** — to access the Veryfi API (Client ID, API key, username, URL)

Steps:

* If you don’t have a Veryfi account, register here: [https://app.veryfi.com/signup/api/](https://app.veryfi.com/signup/api/)

* In the Veryfi hub, go to **`Settings` → `Keys`**:

  * **API Auth Credentials** — copy your API credentials.
  * **Lens: Maven (Android)** — create Maven credentials.

* Add your Maven credentials to your system environment (e.g., `~/.zshrc`):

  ```bash
  export MAVEN_VERYFI_USERNAME=[USERNAME]
  export MAVEN_VERYFI_PASSWORD=[PASSWORD]
  ```

* Test your credentials:

  ```bash
  curl -sS --head \
  https://$MAVEN_VERYFI_USERNAME:$MAVEN_VERYFI_PASSWORD@nexus.veryfi.com/repository/maven-public/com/veryfi/lens/veryfi-lens-sdk/2.1.0.25/veryfi-lens-sdk-2.1.0.25.pom \
  | grep "HTTP/2"
  ```

  * **`HTTP/2 200`** — credentials are valid
  * **`HTTP/2 401`** — invalid credentials. Check your environment variables or create new ones in the Veryfi hub. To verify variables:

    ```bash
    echo $MAVEN_VERYFI_USERNAME
    echo $MAVEN_VERYFI_PASSWORD
    ```

---

#### **2. Installation**

To install the Veryfi Lens SDK, update the following:

1. **Add the private Maven repository** to `settings.gradle` or `settings.gradle.kts`

   * Groovy (`settings.gradle`):

     ```
dependencyResolutionManagement {
    repositories {
        ...
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
   * Kotlin (`settings.gradle.kts`):

     ```
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        ...
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

2. **Add `noCompress` property** to `app/build.gradle` or `app/build.gradle.kts`

   * Groovy:

     ```
android {
    ...
    androidResources {
        noCompress += 'veryfi'
    }
}
     ```
   * Kotlin:

     ```
android {
    ...
    androidResources {
        noCompress += "veryfi"
    }
}
     ```

3. **Add SDK dependency** (replace `VERYFI_SDK_VERSION` with the latest version from [releases](https://github.com/veryfi/veryfi-lens-receipts-android-demo/releases))

   * Groovy:

     ```
dependencies {
    implementation "com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION"
    ...
}
     ```
   * Kotlin:

     ```
dependencies {
    implementation("com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION")
    ...
}
     ```

4. **Update `AndroidManifest.xml`**

   * If `android:allowBackup` or `android:usesCleartextTraffic` are present, add the `tools` namespace and update the `<application>` tag:

     ```
<manifest xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools">
<application
    ...
    tools:ignore="AllowBackup,GoogleAppIndexingWarning"
    tools:replace="android:allowBackup, android:usesCleartextTraffic">
    ...
</manifest>
     ```

5. **Sync Project** in Android Studio:
   `File → Sync Project with Gradle Files`

---

#### **3. Initialization**

1. **Import required Lens SDK classes**:

   ```
import com.veryfi.lens.VeryfiLens
import com.veryfi.lens.VeryfiLensDelegate
import com.veryfi.lens.helpers.DocumentType
import com.veryfi.lens.helpers.VeryfiLensCredentials
import com.veryfi.lens.helpers.VeryfiLensSettings
   ```

2. **Set authentication credentials** (replace placeholders with values from step 1):

   ```
val veryfiLensCredentials = VeryfiLensCredentials()
veryfiLensCredentials.clientId = "CLIENT_ID"  // replace XXX with your Client Id
veryfiLensCredentials.username = "USERNAME"  // replace XXX with your username
veryfiLensCredentials.apiKey = "API_KEY"    // replace XXX with your API Key
veryfiLensCredentials.url = "URL"       // replace with your API environment URL
   ```

3. **Configure Lens settings** ([Settings documentation](https://docs.veryfi.com/lens/mobile/settings/)):

   ```
val veryfiLensSettings = VeryfiLensSettings()
veryfiLensSettings.autoRotateIsOn = true
veryfiLensSettings.autoSubmitDocumentOnCapture = true
veryfiLensSettings.documentTypes = arrayListOf(DocumentType.RECEIPT)
veryfiLensSettings.galleryIsOn = false
veryfiLensSettings.moreMenuIsOn = false
   ```

4. **Implement `VeryfiLensDelegate`** to handle events:

   ```
class MyActivity: VeryfiLensDelegate {

    override fun veryfiLensClose(json: JSONObject) {
        // Process the JSON response here
    }

    override fun veryfiLensError(json: JSONObject) {
        // Process the JSON response here
    }

    override fun veryfiLensSuccess(json: JSONObject) {
        // Process the JSON response here
    }

    override fun veryfiLensUpdate(json: JSONObject) {
        // Process the JSON response here
    }

}
   ```

5. **Register the delegate**:

   ```
VeryfiLens.setDelegate(this)
   ```

6. **Initialize the SDK** with credentials, settings, and your `Application` instance:

   ```
VeryfiLens.configure(application, credentials, settings) { }
   ```

---

#### **4. Starting the Lens (Camera) Flow**

1. **Launch the Lens camera**:

   ```
VeryfiLens.showCamera()
   ```

2. **Process the result** in:

   ```kotlin
   VeryfiLensDelegate.veryfiLensSuccess(json)
   ```

---

#### **5. Full Example: `MyActivity.kt`**

```
import android.util.Log
import com.veryfi.lens.VeryfiLens
import com.veryfi.lens.VeryfiLensDelegate
import com.veryfi.lens.helpers.DocumentType
import com.veryfi.lens.helpers.VeryfiLensCredentials
import com.veryfi.lens.helpers.VeryfiLensSettings

class MyActivity : AppCompatActivity(), VeryfiLensDelegate {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val veryfiLensCredentials = VeryfiLensCredentials()
        veryfiLensCredentials.clientId = "CLIENT_ID"  // replace XXX with your Client Id
        veryfiLensCredentials.username = "USERNAME"  // replace XXX with your username
        veryfiLensCredentials.apiKey = "API_KEY"    // replace XXX with your API Key
        veryfiLensCredentials.url = "URL"       // replace with your API environment URL

        val veryfiLensSettings = VeryfiLensSettings()
        veryfiLensSettings.autoCaptureIsOn = true
        veryfiLensSettings.autoRotateIsOn = true
        veryfiLensSettings.documentTypes = arrayListOf(DocumentType.RECEIPT)

        VeryfiLens.setDelegate(this)

        VeryfiLens.configure(application, credentials, settings) { }
    }

    override fun veryfiLensClose(json: JSONObject) {
        Log.d("MyActivity", json.toString(2))
    }

    override fun veryfiLensError(json: JSONObject) {
        Log.d("MyActivity", json.toString(2))
    }

    override fun veryfiLensSuccess(json: JSONObject) {
        Log.d("MyActivity", json.toString(2))
    }

    override fun veryfiLensUpdate(json: JSONObject) {
        Log.d("MyActivity", json.toString(2))
    }

    private fun onSomeButtonClicked() {
        VeryfiLens.showCamera()
    }
}
```
