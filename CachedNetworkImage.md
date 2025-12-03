CachedNetworkImage는 Flutter에서 **네트워크 이미지를 디스크 + 메모리에 캐싱**해주는 라이브러리야.

네가 지금 TaskDetailPage에서 이미지 썸네일을 불러올 때 사용하는 패키지이기도 하고, 이미지 캐싱을 제대로 하려면 사실상 **업계 표준**으로 쓰는 방식임.

  

전체 개념, 구조, 동작 방식(원리), 내부 캐시 메커니즘까지 실제 엔진 레벨로 정확히 설명해줄게.

---

# **📌 1. CachedNetworkImage란?**

  

Flutter 커뮤니티에서 가장 많이 쓰는 이미지 캐싱 위젯.

```
CachedNetworkImage(
  imageUrl: url,
  fit: BoxFit.cover,
)
```

를 쓰기만 하면,

- 네트워크에서 이미지 다운로드
    
- 디스크(파일로) 저장
    
- 메모리에 저장
    
- 다음번 요청 시 **다운로드 없이 캐시된 이미지 표시**
    
- 스크롤 시 재다운로드 없음
    
- 앱 재시작해도 캐시 유지
    

  

까지 **전부 자동**으로 처리함.

---

# **📌 2. 이걸 왜 써야 하는가?**

  

Image.network() 는 기본적으로:

- **메모리 캐싱만** 함
    
- 화면에서 사라졌다 다시 보이면 다시 다운로드됨
    
- 앱 재시작 시 무조건 다시 다운로드됨
    

  

즉 **네트워크 비용·로딩 시간이 계속 반복됨**.

  

하지만 CachedNetworkImage 는:

- 디스크에 파일 저장
    
- 재다운로드 방지
    
- 스크롤 중 지연 없음
    
- Supabase 트래픽 비용 감소
    
- 체감 UX 상승
    

  

OSSU처럼 **이미지 많은 앱에서는 필수**야.

---

# **📌 3. CachedNetworkImage의 동작 원리**

  

### **핵심 구조는 3가지**

---

## **① CacheManager가 실제 캐시 파일을 관리한다**

  

cached_network_image는 내부적으로 flutter_cache_manager를 사용해.

  

즉,

- CacheManager가 이미지 URL을 key로 삼아
    
- 디스크에 파일을 저장하고
    
- 파일 존재 여부 체크하고
    
- 캐시 유효기간 체크하고
    
- 만료 시 삭제하고
    
- 요청 시 파일을 읽어서 ImageProvider로 넘겨줌
    

  

구조는 이렇게 생김:

```
CachedNetworkImage
   ↓
ImageProvider (CachedNetworkImageProvider)
   ↓
CacheManager (DefaultCacheManager)
   ↓
FileSystem (디스크에 파일 저장)
   ↓
Memory Cache (LRU)
```

---

## **② 네트워크 요청은 오직 최초 1회만 함**

  

동일한 URL이 10번 나타나도:

```
이미 디스크에 캐시 있음 → 바로 파일 로드 → 디코딩 → 이미지 표시
```

네트워크 요청은 아예 발생하지 않음.

---

## **③ 디스크 캐시 + 메모리 캐시 (LRU)**

  

CachedNetworkImage는 두 가지 캐시를 동시에 사용해:

  

### **1️⃣ Disk Cache**

- 실제 파일로 저장
    
- 기기 파일 시스템에 보관
    
- 앱 재시작해도 유지됨
    
- 기본적으로 7일(플러그인 내부 기본값) 보관
    

  

### **2️⃣ Memory Cache (LRU 방식)**

- 빠르게 화면에 표시하기 위해 디코딩된 이미지 객체를 메모리에 저장
    
- LRU(Least Recently Used) 알고리즘 사용
    
- 메모리 부족하면 오래된 캐시 제거됨
    

  

즉 “동일한 이미지를 스크롤할 때 로딩 없이 즉시 표시되는 것”은 메모리 캐시 덕분.

---

# **📌 4. CachedNetworkImage가 실제로 화면을 표시하는 흐름**

  

이미지 로딩 과정을 실제적으로 표현하면:

---

## **1단계: build() → CachedNetworkImage 등장**

  

Flutter 위젯 트리가 CachedNetworkImage를 호출

---

## **2단계: CachedNetworkImageProvider 가 URL을 key로 캐시 조회**

```
CacheManager.getFileFromCache(url)
```

→ 디스크에 파일 있는지 확인

---

## **3단계: 디스크 캐시가 있으면 바로 로딩**

  

파일이 있다면 즉시 불러서 디코딩한다:

```
File → ByteData → Codec → Image → ui.Image
```

→ 화면에 즉시 그림

---

## **4단계: 디스크 캐시가 없다면 네트워크 다운로드**

  

다운로드 후:

- 디스크에 저장
    
- 메모리에도 저장
    
- 이미지 디코딩해서 표시
    

---

## **5단계: 이후 동일 URL 요청은 디스크 캐시만 사용**

  

기기 네트워크가 끊겨 있어도 표시됨.

---

# **📌 5. 캐싱은 어떻게 삭제/갱신되는가?**

  

기본 캐싱 정책은:

- **stalePeriod: 7일**
    
- **maxNrOfCacheObjects: 200개**
    
- 오래된 캐시부터 LRU 규칙으로 삭제됨
    

  

원하면 정책 조절 가능함:

```
class OssuCacheManager extends CacheManager {
  OssuCacheManager()
      : super(Config(
          'ossuCache',
          stalePeriod: Duration(days: 30),
          maxNrOfCacheObjects: 400,
        ));
}
```

그리고 사용:

```
CachedNetworkImage(
  imageUrl: url,
  cacheManager: OssuCacheManager(),
)
```

---

# **📌 6. Supabase Signed URL과 캐싱 문제?**

  

너가 지금 쓰는 URL은 모두 “signedUrl” (일부 시간 후 만료되는 URL)임.

  

예:

```
...?token=abcd...&exp=1764836006
```

### **✔️ 이 경우 캐싱 가능함 (유효기간 동안만)**

  

왜냐하면:

- token 값이 달라지면 다른 URL로 인식돼서 새로 캐시함
    
- 같은 token이면 항상 캐시를 재사용함
    
- Supabase token 만료되면 다시 fetch해야 하는 구조
    

---

# **📌 7. CachedNetworkImage가 OSSU에 적합한 이유**

  

OSSU의 Task/Chat/Board/Media 업로드 구조는:

- 사진/동영상 첨부가 매우 많음
    
- 미디어 리스트 가로 스크롤이 항상 등장
    
- supabase signed url 요청 비용 큼
    
- 사진 재다운로드 발생 시 UX 저하 큼
    

  

CachedNetworkImage를 쓰면:

- 빠른 스크롤 → 이미지 즉시 표시
    
- 트래픽 절감
    
- 로딩 스피너 거의 안 뜸
    
- Supabase 비용 절감
    
- UX 드라마틱하게 개선됨
    

---

# **📌 8. 간단한 사용 예제**

```
CachedNetworkImage(
  imageUrl: url,
  placeholder: (context, url) => CircularProgressIndicator(),
  errorWidget: (context, url, error) => Icon(Icons.error),
)
```

너의 코드에서는 errorWidget만 사용 중:

```
CachedNetworkImage(
  imageUrl: url,
  fit: BoxFit.cover,
  errorWidget: (context, url, error) => Container(
    color: AppColors.objectTertiary,
    child: const Icon(Icons.broken_image),
  ),
)
```

---

# **📌 9. 네 코드에서 CachedNetworkImage가 하는 일 요약:**

  

✔ 첨부 이미지 썸네일을 디스크 캐시로 저장

✔ 스크롤할 때 네트워크 요청 없이 즉시 표시

✔ 이전에 본 항목이면 완전 무지연 표시

✔ Supabase signed URL도 그대로 캐싱됨

✔ 반복 스크롤 시 플리커(깜빡임) 없음

✔ 페이지 이동했다 돌아오면 즉시 표시

  

엄청난 성능 차이가 발생해.

---

# **✨ 원하면?**

- OssuCacheManager 커스텀 정책 적용해줄게
    
- Video thumbnail도 캐싱되는 버전으로 변경 가능
    
- 전체 이미지 캐시 용량/기간 조절 세팅 추가 가능
    
- DetailTaskPage 전체 리팩토링(썸네일·캐싱·정렬 개선)도 해줄 수 있음
    

  

말해줘!