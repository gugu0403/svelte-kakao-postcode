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
