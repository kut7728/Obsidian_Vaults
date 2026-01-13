## 수정 내용 설명

### 문제
iOS에서 `permission_handler` 패키지를 사용할 때, **위치 권한 기능은 기본적으로 비활성화**되어 있습니다. 이를 활성화하려면 Podfile에 매크로를 명시적으로 설정해야 합니다.

이 설정이 없으면:
- 위치 권한을 요청해도 시스템 팝업이 안 뜸
- `serviceStatus`가 항상 `disabled`로 반환됨
- 설정 앱의 위치 서비스 목록에 앱이 나타나지 않음

### 수정한 부분

`ios/Podfile`의 `post_install` 블록에 다음 코드를 추가했습니다:

```67:75:ios/Podfile
      # 배포 타겟 설정
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '14.0'

      # PermissionHandler - 위치 권한 활성화
      config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= [
        '$(inherited)',
        'PERMISSION_LOCATION=1',
      ]
```

### 이 코드가 하는 일

`PERMISSION_LOCATION=1` 매크로는 `permission_handler` 패키지에게 **"이 앱은 위치 권한을 사용할 거야"**라고 알려줍니다. 

이 설정이 있어야:
1. 위치 권한 관련 네이티브 코드가 컴파일됨
2. `Permission.locationWhenInUse.request()` 호출 시 시스템 권한 팝업이 나타남
3. 설정 앱의 위치 서비스 목록에 앱이 표시됨

---

이제 앱을 삭제 → 클린 빌드 → 재설치하면 위치 권한 요청이 정상 작동할 거예요!