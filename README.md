# react-native-kordoc
서명된 APK 
Android는 모든 앱을 설치하기 전에 인증서로 디지털서명을 해야한다. 그래서 [Google Play 스토어](https://play.google.com/store) 를 통해 너의 Android 애플리케이션을 배포하려면 서명된 release APK를 생성해야한다. Android 개발 문서의 [응용프로그램 서명 페이지](https://developer.android.com/studio/publish/app-signing.html)에서 주제에 대해 자세히 얘기한다. 이 가이드에서는 프레세스를 간단히 설명하고, JavaScript 번들을 패키징하는데 필요한 단계를 나열한다.

# 서명 키 생성
너는 keytool을 사용하여 비공개 서명키를 생성할 수 있다. 윈도우는 keytool은 `C:\Program Files\Java\jdkx.x.x_x\bin`에서 실행해야한다.

> $ keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000

이 명령프롬프트는 키 저장소와 키의 암호를 묻는 메시지와 키의 고유 이름 필드를 제공한다. 그런 다음 keystore를 'my-release-key.keystore' 라는 파일로 생성합니다.

keystore에는 10000일 동안 유효한 서명키가 포함되어 있다. 별칭은 나중에 앱에 서명할 때 사용할 이름이다. 그래서 별칭을 기록해두는 것을 기억해라.

Note: 너의 keystore 파일을 비공개로 유지하고 그것을 버전관리에 절대 커밋하지마라.

## gredle 변수 설정
1.너의 프로젝트의 `android/app` 폴더 하위 디렉토리에 `my-release-key.keystore` 파일을 놓는다.
2.`~/.gradle/gradle.properties` 파일을 수정하고 다음을 추가한다. (*****를 올바른 키 keystore 비밀번호, 별칭 및 키 비밀번호로 대체하십시오.).

> MYAPP_RELEASE_STORE_FILE=my-release-key.keystore  
MYAPP_RELEASE_KEY_ALIAS=my-key-alias  
MYAPP_RELEASE_STORE_PASSWORD=*****  
MYAPP_RELEASE_KEY_PASSWORD=*****  

이것들은 글로벌 gradle 변수가 될것이다. 우리는 우리의 응용 프로그램에 서명하기 위해 나중에 gradle 구성에서 사용할 수 있습니다.

> keystore 저장에 대한 참고사항:  
플레이스토어에 앱을 게시 한 후 언제든지 서명 키를 변경하려면 앱을 다른 패키지 이름 (모든 다운로드 및 등급 손실)으로 다시 게시해야합니다. 따라서 keystore를 백업하고 암호를 잊어 버리지 마십시오.

보안 참고 사항: 암호를 일반 텍스트로 저장하는 것을 원하지 않고 OSX를 실행하는 경우 키 [체인 접근 앱에 자격 증명을 저장](https://pilloxa.gitlab.io/posts/safer-passwords-in-gradle/)할 수도 있다. 그런 다음 `~/.gradle/gradle.properties` 의 마지막 두 행을 건너 뛸 수 있습니다.

## 앱의 gradle 설정에 서명config 추가
프로젝트 폴더의 `android/app/build.gradle` 파일을 수정하고 서명config를 추가한다.

```
...
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
...
```
## release APK 생성

터미널에서 다음을 실행하기만 하면 된다:
> $ cd android && ./gradlew assembleRelease

Gradle의 `assembleRelease`는 앱을 실행하는 데 필요한 모든 JavaScript를 APK에 번들합니다. JavaScript 번들 및/또는 drawable resources가 번들되는 방식을 변경해야하는 경우 (예 : 기본 파일 / 폴더 이름 또는 프로젝트의 일반 구조를 변경 한 경우), `android / app / build.gradle` 을 보고 이러한 변경 사항을 반영하여 업데이트 할 수있는 방법을 확인하십시오.

생성 된 APK는 `android/app/build/outputs/apk/app-release.apk` 에서 찾을 수 있다, 그리고 배포 할 준비가 되어있다.

## 앱의 출시 빌드 테스트

release 빌드를 플레이스토어에 업로드 하기 전에 철처하게 테스트 해야한다. 다음을 사용하여 장치에 설치:
> $ react-native run-android --variant=release

`--variant = release`는 위에서 설명한대로 서명을 설정 한 경우에만 사용할 수 있습니다.

너는 실행중인 다른 패키지 인스턴스를 삭제 할 수 있고, 모든 프레임워크 자바스크립트 코드는 APK의 번들로 제공된다.

## Proguard를 활성화하여 APK 크기 줄이기 (선택 사항)

Proguard는 APK의 크기를 약간 줄일 수 있는 도구이다. 이는 앱이 사용하지 않는 React Native Java bytecode (및 의존성)의 일부를 제거하여 수행한다.

**중요**: Proguard를 사용하는 경우 앱을 철처히 테스트해야한다. Proguard는 종종 각 네이티브 라이브러리에 구체적인 구성을 필요로합니다. `app / proguard-rules.pro`를 참조하십시오.

Proguard를 사용하려면, `android/app/build.gradle`를 수정해라.

```
/**
 * Run Proguard to shrink the Java bytecode in release builds.
 */
def enableProguardInReleaseBuilds = true
```
