# 카카오 우편번호 검색

Svelte용 카카오 우편번호 검색 콤포넌트 입니다.
kakao_postcode.svelte를 개발하는 프로젝트의 components 디렉토리 등에 복사 후 사용해 주세요.

```
<KakaoPostCode
  id="엘리먼트 아이디"
  on:select={우편번호 선택 시 실행할 함수}
  style="우편번호 검색 창 사용자 스타일"
  imgClose="닫기버튼 이미지 URL"
  imgCloseStyle="닫기버튼 스타일"
  hideCloseBtn
>
</KakaoPostCode>
```

```
<KakaoPostCode
  id="search_postcode"
  on:select={choosePostCode}
  style="border:none; width:100%; height:100%; max-width:100%; max-height:100%;"
  hideCloseBtn
>
</KakaoPostCode>
```

## 샘플코드
```
<main>
  <div>
    <input type="text" id="postcode" name="postcode" value={postcode} />
    <button on:click={searchZipCode}>우편번호검색</button>
  </div>
  <div>
    <input type="text" id="address" name="address" value={address} style="width:500px;" /><br>
  </div>

  <KakaoPostCode id="search_postcode" on:select={choosePostCode}></KakaoPostCode>
</main>

<script>
import KakaoPostCode from './kakao_postcode.svelte';

let postcode = '';
let address = '';

function searchZipCode() {
  document.getElementById('search_postcode').style.display = 'block';
}

function choosePostCode(info) {
  postcode = info.detail.postcode;
  address = info.detail.address;
}
</script>
```
