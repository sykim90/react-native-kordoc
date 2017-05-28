# react-native-kordoc
서명된 APK 
Android는 모든 앱을 설치하기 전에 인증서로 디지털서명을 해야한다. 그래서 구글플레이 스토어를 통해 너의 안드로이드 애플리케이션을 배포하려면 서명된 release APK를 생성해야한다. Android 개발 문서의 응용프로그램 서명 페이지에서 주제에 대해 자세히 얘기한다. 이 가이드에서는 프레세스를 간단히 설명하고, JavaScript 번들을 패키징하는데 필요한 단계를 나열한다.

서명 키 생성
너는 keytool을 사용하여 비공개 서명키를 생성할 수 있다. 윈도우는 keytool은 `C:\Program Files\Java\jdkx.x.x_x\bin`에서 실행해야한다.

> $ keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000

이 명령프롬프트는 키 저장소와 키의 암호를 묻는 메시지와 키의 고유 이름 필드를 제공한다. 그런 다음 keystore를 'my-release-key.keystore' 라는 파일로 생성합니다.

keystore에는 10000일 동안 유효한 서명키가 포함되어 있다. 별칭은 나중에 앱에 서명할 때 사용할 이름이다. 그래서 별칭을 기록해두는 것을 기억해라.

Note: 너의 keystore 파일을 비공개로 유지하고 그것을 버전관리에 절대 커밋하지마라.

gredle 변수 설정
1.너의 프로젝트의 'android/app' 폴더 하위 디렉토리에 'my-release-key.keystore' 파일을 놓는다.
2.'~/.gradle/gradle.properties' 파일을 수정하고 다음을 추가한다. (*****를 올바른 키 keystore 비밀번호, 별칭 및 키 비밀번호로 대체하십시오.).

> MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
MYAPP_RELEASE_KEY_ALIAS=my-key-alias
MYAPP_RELEASE_STORE_PASSWORD=*****
MYAPP_RELEASE_KEY_PASSWORD=*****

이것들은 글로벌 gradle 변수가 될것이다. 우리는 우리의 응용 프로그램에 서명하기 위해 나중에 gradle 구성에서 사용할 수 있습니다.

>> Note about saving the keystore:
Once you publish the app on the Play Store, you will need to republish your app under a different package name (losing all downloads and ratings) if you want to change the signing key at any point. So backup your keystore and don't forget the passwords.
Note about security: If you are not keen on storing your passwords in plaintext and you are running OSX, you can also store your credentials in the Keychain Access app. Then you can skip the two last rows in ~/.gradle/gradle.properties.

Adding signing config to your app's gradle config 
Edit the file android/app/build.gradle in your project folder and add the signing config,

```
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
                storeFile file(MYAPP_RELEASE_STORE_FILE)
                storePassword MYAPP_RELEASE_STORE_PASSWORD
                keyAlias MYAPP_RELEASE_KEY_ALIAS
                keyPassword MYAPP_RELEASE_KEY_PASSWORD
            }
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
```
Generating the release APK 
Simply run the following in a terminal:

$ cd android && ./gradlew assembleRelease
Gradle's assembleRelease will bundle all the JavaScript needed to run your app into the APK. If you need to change the way the JavaScript bundle and/or drawable resources are bundled (e.g. if you changed the default file/folder names or the general structure of the project), have a look at android/app/build.gradle to see how you can update it to reflect these changes.

The generated APK can be found under android/app/build/outputs/apk/app-release.apk, and is ready to be distributed.

Testing the release build of your app 
Before uploading the release build to the Play Store, make sure you test it thoroughly. Install it on the device using:

$ react-native run-android --variant=release
Note that --variant=release is only available if you've set up signing as described above.

You can kill any running packager instances, all your and framework JavaScript code is bundled in the APK's assets.

Enabling Proguard to reduce the size of the APK (optional) 
Proguard is a tool that can slightly reduce the size of the APK. It does this by stripping parts of the React Native Java bytecode (and its dependencies) that your app is not using.

IMPORTANT: Make sure to thoroughly test your app if you've enabled Proguard. Proguard often requires configuration specific to each native library you're using. See app/proguard-rules.pro.

>To enable Proguard, edit android/app/build.gradle:
```
/**
 * Run Proguard to shrink the Java bytecode in release builds.
 */
def enableProguardInReleaseBuilds = true
```
