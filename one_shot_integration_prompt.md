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
   - Get the latest version number from https://github.com/veryfi/veryfi-lens-receipts-android-demo/releases/latest.
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
