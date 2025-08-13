### **Veryfi Lens for Mobile — LLM-Friendly Cookbook**

#### **Purpose**

This guide provides precise steps to integrate the **Veryfi Lens for Mobile Android SDK** into an Android app. It is optimized for use with AI coding assistants such as Cursor AI or Claude Code.

---

### **Prerequisites**

* API credentials and Maven credentials from Veryfi ([Authentication guide](https://docs.veryfi.com/lens/mobile/introduction/authentication/))
* Kotlin or Java Android project
* Gradle version **8+** and Java **17+**
* Minimum Android SDK version **23** (Android 6.0)

---

## **1. Obtain Credentials**

1. Create a Veryfi account:
   `https://app.veryfi.com/signup/api/`

2. In the Veryfi Hub, go to:
   `Settings → Keys`

   * Copy API credentials from **API Auth Credentials**
   * Create Maven credentials from **Lens: Maven (Android)**

3. Save Maven credentials as environment variables:

```bash
   export MAVEN_VERYFI_USERNAME=<USERNAME>
   export MAVEN_VERYFI_PASSWORD=<PASSWORD>
```

4. Test Maven credentials:

```bash
   curl -sS --head \
   https://$MAVEN_VERYFI_USERNAME:$MAVEN_VERYFI_PASSWORD@nexus.veryfi.com/repository/maven-public/com/veryfi/lens/veryfi-lens-sdk/2.1.0.25/veryfi-lens-sdk-2.1.0.25.pom \
   | grep "HTTP/2"
```

   * Output **`HTTP/2 200`** = valid
   * Output **`HTTP/2 401`** = invalid → re-check environment variables:

  ```bash
     echo $MAVEN_VERYFI_USERNAME
     echo $MAVEN_VERYFI_PASSWORD
  ```

---

## **2. Install the SDK**

1. Add the private Maven repository to **settings.gradle** (Groovy):

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

   or **settings.gradle.kts** (Kotlin):

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

2. In **app/build.gradle** (Groovy) or **app/build.gradle.kts** (Kotlin), add the `noCompress` property:

```
android {
    ...
    androidResources {
        noCompress += "veryfi"
    }
}
```

3. Add SDK dependency (replace `<VERYFI_SDK_VERSION>` with latest from [releases](https://github.com/veryfi/veryfi-lens-receipts-android-demo/releases)):

```
dependencies {
    implementation("com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION")
    ...
}
```

4. Update `AndroidManifest.xml`:

   * If `android:allowBackup` or `android:usesCleartextTraffic` are present, add the `tools` namespace and required attributes to `<application>`:

  ```
<manifest xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools">
<application
    ...
    tools:ignore="AllowBackup,GoogleAppIndexingWarning"
    tools:replace="android:allowBackup, android:usesCleartextTraffic">
    ...
</manifest>
  ```

5. Sync Gradle in Android Studio:
   **Menu → File → Sync Project with Gradle Files**

---

## **3. Initialize the SDK**

1. Import Lens SDK classes:

```
import com.veryfi.lens.VeryfiLens
import com.veryfi.lens.VeryfiLensDelegate
import com.veryfi.lens.helpers.DocumentType
import com.veryfi.lens.helpers.VeryfiLensCredentials
import com.veryfi.lens.helpers.VeryfiLensSettings
```

2. Configure API credentials (replace placeholders with API Auth Credentials):

```
val veryfiLensCredentials = VeryfiLensCredentials()
veryfiLensCredentials.clientId = "CLIENT_ID"  // replace XXX with your Client Id
veryfiLensCredentials.username = "USERNAME"  // replace XXX with your username
veryfiLensCredentials.apiKey = "API_KEY"    // replace XXX with your API Key
veryfiLensCredentials.url = "URL"       // replace with your API environment URL
```

3. Configure Lens settings ([Settings documentation](https://docs.veryfi.com/lens/mobile/settings/)):

```
val veryfiLensSettings = VeryfiLensSettings()
veryfiLensSettings.autoRotateIsOn = true
veryfiLensSettings.autoSubmitDocumentOnCapture = true
veryfiLensSettings.documentTypes = arrayListOf(DocumentType.RECEIPT)
veryfiLensSettings.galleryIsOn = false
veryfiLensSettings.moreMenuIsOn = false
```

4. Implement `VeryfiLensDelegate` to handle SDK events:

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

5. Register the delegate:

```
VeryfiLens.setDelegate(this)
```

6. Initialize the SDK with credentials, settings, and `Application` instance:

```
VeryfiLens.configure(application, credentials, settings) { }
```

---

## **4. Start the Lens Camera Flow**

1. Launch the Lens camera:

```
VeryfiLens.showCamera()
```

2. Handle success in:

```kotlin
   VeryfiLensDelegate.veryfiLensSuccess(<JSON_RESULT>)
```

---

## **5. Full Activity Example**

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
