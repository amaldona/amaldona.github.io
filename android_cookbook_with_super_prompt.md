### **Veryfi Lens for Mobile — Cookbook for Cursor AI / Claude Code Integration**

#### **Overview & Purpose**

In this cookbook you will learn how to integrate the **Veryfi Lens for Mobile Android SDK** into Android apps—particularly via LLMs like Cursor AI or Claude Code.

#### **Prerequisites**

- API and Maven credentials (more info: https://docs.veryfi.com/lens/mobile/introduction/authentication/)
- Kotlin or Java project with Gradle 8 and Java 17 minimum
- Minimum Android SDK 6.0 (API Level 23)    
    
#### **1. Obtaining credentials**

- You need two sets of credentials:

  - Maven credentials to download the SDK (Maven username and password)
  - Access credentials to access to the Veryfi API (Client ID, API key, username, URL)

- If you don't have an account with Veryfi, please register here:  [https://app.veryfi.com/signup/api/](https://app.veryfi.com/signup/api/)

- In the Veryfi hub, go to  `Settings`  >  `Keys`:
  - In the `API Auth Credentials` section, you can get your API credentials.
  - In the `Lens: Maven (Android)`  section you can create your Maven credentials.

- Add your Maven credentials to your system environment (e.g.,  `~/.zshrc`). Replace  `[MAVEN_USERNAME]`  and  `[MAVEN_PASSWORD]`  with the credentials obtained in the previous step.

  ```
  export MAVEN_VERYFI_USERNAME=[USERNAME]
  export MAVEN_VERYFI_PASSWORD=[PASSWORD]
  ```

- To check if the credentials are correct, run the next command in the terminal:

```
curl -sS --head https://$MAVEN_VERYFI_USERNAME:$MAVEN_VERYFI_PASSWORD@nexus.veryfi.com/repository/maven-public/com/veryfi/lens/veryfi-lens-sdk/2.1.0.25/veryfi-lens-sdk-2.1.0.25.pom | grep "HTTP/2"
```

  - If prints `HTTP/2 200`, then the credentials are valid
  - but if it is `HTTP/2 401`, the credentials are not valid. Please double check the environment variables, or create new credentials in the Veryfi hub `Keys` page. To check the environment variables' values, run the next command to print them in the terminal:

  ```
  echo $MAVEN_VERYFI_USERNAME
  echo $MAVEN_VERYFI_PASSWORD
  ```


#### **2. Installation**

- To install the Veryfi Lens SDK for Android, you need to modify the next files:
  - Add the Veryfi Maven repository at https://nexus.veryfi.com/repository/maven-releases/ to `settings.gradle` and use the environment variables `MAVEN_VERYFI_USERNAME` and `MAVEN_VERYFI_PASSWORD` for authentication.
  - Add the `noCompress` property and the `implementation` dependency to `app/build.gradle`
  - Modify the `<application>` tag attributes in `AndroidManifest.xml`

- Register the private Maven repository in the project-level `settings.gradle` or `settings.gradle.kts`:

  - `settings.gradle` (Groovy):
  
```
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

  - `settings.gradle.kts` (Kotlin):
  
```
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

- In your `app/build.gradle` or `app/build.gradle.kts` file, add the `noCompress` property:


  - `app/build.gradle` (Groovy):
  
```
android {
    androidResources {
        noCompress += 'veryfi'
    }
}
```

  - `app/build.gradle.kts` (Kotlin):

```
android {
    androidResources {
        noCompress += "veryfi"
    }
}
```

- Add the dependencies. Replace VERYFI_SDK_VERSION with the latest version (Versions and release notes: https://github.com/veryfi/veryfi-lens-receipts-android-demo/releases):

  - `app/build.gradle` (Groovy):

```
dependencies {
    implementation "com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION"
}
```

  - `app/build.gradle.kts` (Kotlin):

```
dependencies {
    implementation("com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION")
}
```

- If your `AndroidManifest.xml` has `android:allowBackup` tag, add the namespace `tools` to the manifest, and include the following settings to the `<application>` tag:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools">
    <application
        tools:ignore="AllowBackup,GoogleAppIndexingWarning"
        tools:replace="android:allowBackup">
    </application>
</manifest>
```

- Finally, if you are using Android Studio, select `File` -> `Sync Project with Gradle Files` to download the depedencies and refresh the internal project model.


#### **3. Initialization**

-  Import required classes from Lens SDK:

```kotlin
import com.veryfi.lens.VeryfiLens
import com.veryfi.lens.VeryfiLensDelegate
import com.veryfi.lens.helpers.DocumentType
import com.veryfi.lens.helpers.VeryfiLensCredentials
import com.veryfi.lens.helpers.VeryfiLensSettings
```

- Configure your authentication credentials. Replace `CLIENT_ID`, `USERNAME`, `API_KEY` and `URL` with the `API Auth Credentials` from the step 1:

```kotlin
val credentials = VeryfiLensCredentials().apply {
    clientId = "CLIENT_ID"
    username = "USERNAME"
    apiKey = "API_KEY"
    url = "URL"
}
```

- Configure your Veryfi Lens settings (Settings documentation: https://docs.veryfi.com/lens/mobile/settings/)

```kotlin
val settings = VeryfiLensSettings().apply {
    autoRotateIsOn = true
    autoSubmitDocumentOnCapture = true
    documentTypes = arrayListOf(DocumentType.RECEIPT)
    galleryIsOn = false
    moreMenuIsOn = false
}
```

- Implement the `VeryfiLensDelegate` interface to handle the events triggered by Veryfi Lens:

```kotlin
class MyActivity : VeryfiLensDelegate {
    override fun veryfiLensClose(json: JSONObject) { /* handle */ }
    override fun veryfiLensError(json: JSONObject) { /* handle */ }
    override fun veryfiLensSuccess(json: JSONObject) { /* handle */ }
    override fun veryfiLensUpdate(json: JSONObject) { /* handle */ }
}
```

- Register the delegate:

```kotlin
VeryfiLens.setDelegate(this)
```

- Initialize the SDK with the credentials, settings and the `Application` instance:

```kotlin
VeryfiLens.configure(application, credentials, settings) { }
```

#### **4. Starting the Lens (Camera) Flow**

- Launch the Lens camera:

```kotlin
VeryfiLens.showCamera()
```

- Process the result in `VeryfiLensDelegate.veryfiLensSuccess(json)`. 

#### **5. Full Example: MyActivity.kt**

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

---
## **Veryfi Lens Android SDK — One-Shot Integration Prompt (For AI Tools)**

```
You are an expert Android SDK assistant.  
Integrate the Veryfi Lens for Mobile Android SDK into a Kotlin Android project with Gradle 8+, Java 17+, and minSdk 23.  
Follow these exact steps and output all required code/config changes.

1. **Gradle Configuration**
   - Update settings.gradle to add the Veryfi Maven repository with authentication.
     Example (Groovy):
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
     Example (Kotlin):
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
   - Get the latest version number (without the “v” prefix) from https://github.com/veryfi/veryfi-lens-receipts-android-demo/releases/latest.
   - Add to app/build.gradle and replace VERYFI_SDK_VERSION with the latest version number from the previous step:
     android {
         androidResources {
             noCompress += "veryfi"
         }
     }
     dependencies {
         implementation("com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION")
     }
2. **AndroidManifest Changes**
   - If android:allowBackup exist, add:
     <manifest xmlns:android="http://schemas.android.com/apk/res/android"
               xmlns:tools="http://schemas.android.com/tools">
         <application
             tools:ignore="AllowBackup,GoogleAppIndexingWarning"
             tools:replace="android:allowBackup">
         </application>
     </manifest>

3. **Kotlin Code**
   - Import:
     import com.veryfi.lens.VeryfiLens
     import com.veryfi.lens.VeryfiLensDelegate
     import com.veryfi.lens.helpers.DocumentType
     import com.veryfi.lens.helpers.VeryfiLensCredentials
     import com.veryfi.lens.helpers.VeryfiLensSettings
   - Create credentials:
     val credentials = VeryfiLensCredentials().apply {
         clientId = "CLIENT_ID"
         username = "USERNAME"
         apiKey = "API_KEY"
         url = "URL"
     }
   - Create settings:
     val settings = VeryfiLensSettings().apply {
         autoRotateIsOn = true
         autoSubmitDocumentOnCapture = true
         documentTypes = arrayListOf(DocumentType.RECEIPT)
         galleryIsOn = false
         moreMenuIsOn = false
     }
   - Implement VeryfiLensDelegate with:
     veryfiLensClose(json: JSONObject), veryfiLensError(json: JSONObject), veryfiLensSuccess(json: JSONObject), veryfiLensUpdate(json: JSONObject)
   - Register delegate:
     VeryfiLens.setDelegate(this)
   - Configure SDK:
     VeryfiLens.configure(application, credentials, settings) { }
   - Launch camera:
     VeryfiLens.showCamera()

```