---
layout: post
title: "Flutter 패키지 및 플러그인 개발"
author: "YoonJae"
description: "패키지 및 플러그인 개발 | 초보개발자 기술 블로그"
image: "https://www.google.com/url?sa=i&source=images&cd=&cad=rja&uact=8&ved=2ahUKEwjE4fPjqPLhAhXEzLwKHfkODwQQjRx6BAgBEAU&url=https%3A%2F%2Farstechnica.com%2Fgadgets%2F2018%2F06%2Fgoogles-cross-platform-flutter-sdk-hits-release-preview-1%2F&psig=AOvVaw1lN9eOBIWIit5i_Eoz7AX6&ust=1556524772869984"
---

## Flutter

플러터는 구글에서 만든 프레임워크입니다. RN과 마찬가지로 크로스 플랫폼 개발을 쉽게 할 수 있게 해줍니다. Dart 라는 언어 때문에 고민이 좀 되긴 했는데 일단 구글 님을 믿고 가봅니다.
## React Native와 Flutter 
하이브리드 어플리케이션 화면 UI야 누가 짜든 어찌어찌 하다 보면 모습은 얼추 나옵니다. (성능상 이슈는 구현자에 능력에 따라 다르겠지만) 하지만 없는 네이티브기능이 요구사항으로 요청될 때 내가 이걸 할 줄 모른다면 그때야말로 진짜 치명적인 이슈라고 생각해서 패키지 만드는 법에 대해서 공유해 봅니다. RN을 시작할 때의 빈약했던 문서와 달리 플루터는 친절한 문서를 제공해줍니다.

[플루터 패키지 개발](https://flutter.dev/docs/development/packages-and-plugins/developing-packages) 문서에 보다 친절하게 쓰여 있을 테지만 간단하게 보면

## 패키지 만들기

```dart
  flutter create --org com.example --template=plugin hello
```

위의 명령어를 입력하면 패키지와 example을 포함한 앱 버전을 가져오는 sample 패키지가 생성하게 된다. android나 ios의 언어를 선택하려면

```dart
flutter create --template=plugin -i swift -a kotlin hello
```

위 명령어로 언어를 지정할수있습니다.

## 패키지 구성

```
--패키지 루트
----ios // ios 플랫폼에 필요한 부분을 구현.
----android // android 플랫폼에 필요한 부분을 구현.
----example // 해당 패키지를 이용한 샘플 어플리케이션 구현.
----lib // 내가 작성한 네이티브와 내 어플리케키션을 연결시켜주는 브릿지 부분
----test 
```
프로젝트 구조libs/지정한프로젝트명.dart 이곳이 내가 작성한 네이티브와 내 어플리에키션을 연결시켜주는 브릿지 부분android/ios/에서 각각 플랫폼에 필요한 부분을 구현하면됩니다. 패키지를 만들면 기본적으로 버전을 가져오는 부분이 이미 각각 android ios에 구현이 되있습니다. 그것을 참고해서 작성했습니다. 참고로 너무 간단합니다. RN했을때가 기억은 잘안나지만 RN보다 쉽게 구현한것 같습니다.

## 패키지 구현

실제 프로젝트에서 작성한 패키지를 호출하는 부분

``` dart
import 'package:flutter_naver_login/flutter_naver_login.dart';

NaverLoginResult res = await FlutterNaverLogin.logIn();
```

작성한 패키지 부분

``` dart
class FlutterNaverLogin {

    static const MethodChannel _channel =const MethodChannel'flutter_naver_login');
    static Future<NaverLoginResult> logIn() async {
        final Map<dynamic, dynamic> res = await _channel.invokeMethod('logIn');
        print(res);
        return _delayedToResult(new NaverLoginResult._(res.cast<String, dynamic>()));
    }
```

Kotlin 네이티브 코드 호출 부분

``` kotlin
    override fun onMethodCall(call: MethodCall, result: Result) {
        when (call.method) {
            METHOD_LOG_IN -> this.login(result)
            METHOD_LOG_OUT -> this.logout(result)
            }
    }
```

Kotlin 네이티브 코드 응답 부분

``` kotlin
    result.success(object : HashMap<String, String>() {
        init {
            put("status", "getToken")
            put("accessToken", mOAuthLoginInstance.getAccessToken(mContext))
            put("expiresAt", mOAuthLoginInstance.getExpiresAt(mContext).toString())
            put("tokenType", mOAuthLoginInstance.getTokenType(mContext))
        }
    })
```

위와 같은 형식으로 안드로이드와 ios 부분을 구현해서 example에서 해당 패키지에 관한
샘플 어플리케이션을 구성합니다. 


## 패키지 배포하기

구현 사항이 끝나면 문서를 작성합니다. 플루터는 문서를 중시합니다. 패키지 배포 화면을 보면 publish 돼서 flutter packages에 올라간 패키지들을 문서와 주석 등을 통해서 점수로 그 패키지에 관한 신뢰성을 평가합니다. 또한 기본적인 문서가 없으면 애초에 배포가 불가능합니다.

``` 
 flutter packages pub publish --dry-run
```
명령어를 통해서 빠진 문서에 대한 검사를 할수있습니다. 해당 명령어를 통과할수 있어야 배포가 가능합니다. 해당 명령어를 통과하면 아래 명령어를 통해 배포를 합니다. 

```
  flutter packages pub publish
```

배포하면 심사를 기다리게 되고 보통 1-2시간 안에 심사 중 패키지로 패키지 사이트에 뜨게 되고 하루 안에 new 패키지로 뜨게 되는 거 같습니다.

## 느낀점
- 패키지를 작성하면서 일단 플루터의 개발환경에 대해서 놀랐습니다. 정식 버전이 출시된 기간이 얼마 안 됐음에도 불구하고 개발 도구가 풍부하다고 생각되었습니다. vscode, android studio, intellijs 툴 등에서 빠른 빌드속도를 제공했습니다. 사례나 코드 구현 부분에 대해서는 좀 부족할 수 있지만 아직 생태계가 조성되어가는 과정이라고 생각하고 공식 문서 플루터 document가 작성 돼 있는 게 역시 구글이다 싶었습니다.
- 프론트엔드 개발자가 android, ios 두 플랫폼에 관한 패키지를 작성하다 보니 이번 작성해본 네이버 로그인에 대해서 평가해보자면 카카오나 페이스북 로그인 등등 여러 로그인 sdk 문서를 참조해 봤는데 해당 패키지들은 모두 프로젝트 루트의 xml이나 json에서 해당 어플리케이션의 key값을 관리하는데 네이버는 해당 프로젝트 패키지 내부에서 관리해서 그 부분을 제 입맛에 맞게 수정하느라 안드로이드의 패키지 간의 값을 공유하는 부분을 작성하는 부분에서 많은 부분이 할애돼서 네이티브에 관한 공부도 충분히 되어야 플루터를 같은 멀티플랫폼을 지원하는 프레임워크 사용하는 데 있어서 필수적인 부분이라고 생각되었습니다.


## [플루터 네이버 로그인](https://pub.dartlang.org/packages/flutter_naver_login) 
많이 사용해주시고 이슈 있는 부분이 있으면 등록해주시면 바로 수정하겠습니다.
혹시 잘 사용하신다면 해당 git 프로젝트에 star 눌러주시면 감사하겠습니다. 배포해보니 스타 받으면 매우 뿌듯하더라고요.