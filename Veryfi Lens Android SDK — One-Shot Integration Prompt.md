You are an expert Android SDK assistant.  
Integrate the Veryfi Lens for Mobile Android SDK into a Kotlin Android project with Gradle 8+, Java 17+, and minSdk 23.  
Follow these exact steps and output all required code/config changes.

1. **Credentials**
   - If no Veryfi account: https://app.veryfi.com/signup/api/
   - Get API credentials from Veryfi hub: Settings → Keys → API Auth Credentials (Client ID, Username, API Key, URL).
   - Get Maven credentials from Veryfi hub: Settings → Keys → Lens: Maven (Android) (Maven Username, Maven Password).
   - Store in environment variables:
     export MAVEN_VERYFI_USERNAME=[USERNAME]
     export MAVEN_VERYFI_PASSWORD=[PASSWORD]
   - Verify with:
     curl -sS --head https://$MAVEN_VERYFI_USERNAME:$MAVEN_VERYFI_PASSWORD@nexus.veryfi.com/repository/maven-public/com/veryfi/lens/veryfi-lens-sdk/[VERSION]/veryfi-lens-sdk-[VERSION].pom | grep "HTTP/2"

2. **Gradle Configuration**
   - Add Maven repository to settings.gradle:
     Groovy:
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
   - Add to app/build.gradle:
     android {
         androidResources {
             noCompress += 'veryfi'
         }
     }
     dependencies {
         implementation "com.veryfi.lens:veryfi-lens-sdk:VERYFI_SDK_VERSION"
     }
     (Replace VERYFI_SDK_VERSION with the latest from https://github.com/veryfi/veryfi-lens-receipts-android-demo/releases)

3. **AndroidManifest Changes**
   - If android:allowBackup or android:usesCleartextTraffic exist, add:
     <manifest xmlns:android="http://schemas.android.com/apk/res/android"
               xmlns:tools="http://schemas.android.com/tools">
         <application
             tools:ignore="AllowBackup,GoogleAppIndexingWarning"
             tools:replace="android:allowBackup, android:usesCleartextTraffic">
         </application>
     </manifest>

4. **Kotlin Code**
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
     veryfiLensClose, veryfiLensError, veryfiLensSuccess, veryfiLensUpdate
   - Register delegate:
     VeryfiLens.setDelegate(this)
   - Configure SDK:
     VeryfiLens.configure(application, credentials, settings) { }
   - Launch camera:
     VeryfiLens.showCamera()

5. **Full Example**
   - Output a complete MyActivity.kt that:
     - Extends AppCompatActivity, implements VeryfiLensDelegate
     - Sets credentials & settings in onCreate
     - Implements delegate methods with logging
     - Launches camera on button click

6. **Final Step**
   - In Android Studio: File → Sync Project with Gradle Files
   - Build and run on an Android 6.0+ device.
