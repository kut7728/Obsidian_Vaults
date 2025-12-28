# 개요
- 앱의 특정 화면으로 바로 이동시키는 링크 기술
- 특정 링크를 클릭하면 앱이 켜지면서 설정된 화면으로 바로 이동하는 것


# 종류
## Custom Scheme Link
`myapp://login?token=abc`
- 앱에서 직접 만든 스킴^[[[URL, URI, URN]]]
- 설정도 쉽고, 애플-안드로이드 양 플랫폼 모두 지원되는 장점
- 하지만 앱이 없을경우 웹 fallback이 안됨

``` xml title:info.plist
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLName</key>  // 이 url 스킴 그룹의 이름 (자유)
    <string>com.mycompany.myapp</string>
    <key>CFBundleURLSchemes</key>  // 내가 등록하려는 스킴
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
```

### 처리 절차
1. 사용자가 스킴 링크 클릭
2. OS가 문자열을 보고 “이 스킴을 등록한 앱이 있나?” 확인
3. 앱이 설치됨 → 즉시 앱 실행
4. OS가 링크전체를 앱에 전달
5. 앱의 네이티브 레이어가 링크를 Flutter로 전달
6. Flutter 플러그인 (app_links) 등에서 받아서 파싱
7. 링크 분석후 Navigator, 또는 go_router등으로 특정 페이지 이동

---

## Universal Link (iOS)
`https://myapp.com/deeplink/...`
- Safari를 통해서 앱으로 바로 연결 가능
- 앱이 설치 안되어있을경우 웹으로 fallback
- 웹 fallback을 위해 AASA(apple-app-site-association)파일 필요^[[[Deeplink 적용기]]]

### 처리 절차
1. 사용자가 Https 링크 클릭
2. iOS가 해당 도메인의 AASA파일 확인
	- 앱의 번들Id, 허용된 path 확인
3. 매칭되는 앱이 있다면 해당 앱으로 이동
4. url전체가 앱에 전달됨
5. flutter 앱이 플러그인을 통해 받게됨
6. 페이지 이동

---

## App Link (Android)
- universialLink의 안드로이드 버전 → 형태도 같음
- 웹 fallback을 위해 AssetLinks.json 파일 필요

### 처리 절차
1. 사용자가 Https 링크 클릭
2. 안드로이드가 해당 도메인의 Asset Links 파일 확인
	- 파일속 SHA256과 매칭되는 패키지가 있다면 앱을 호출
	- 없다면 웹으로 fallback
3. url전체가 앱의 intent 에 전달됨
	- intent.getDataSring() 형태로 전달됨
4. flutter 앱이 플러그인을 통해 받게됨
5. 페이지 이동

---

# 딥링크 페이지 라우팅
- go_router같은 외부 라이브러리를 사용해서 간단하게 처리할 수도 있지만
- 없더라도 Navigator.push등으로 라우팅은 가능!