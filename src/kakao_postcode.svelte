<script>
import { createEventDispatcher } from 'svelte';

const dispatch = createEventDispatcher();

/**
 * 카카오 우편번호 core 스크립트 로드 완료여부
 */
let isLoaded = false;

/**
 * useCheckStyle = true 일 경우
 * display style 값을 주기적으로 체크하여 display = block 일 경우 카카오 우편번호 검색 스크립트 실행(new daum.Postcode())
 *
 * useCheckStyle = false 일 경우
 * 초기 스크립트 로딩 후 & 우편번호 검색 창을 닫았을 때 카카오 우편번호 서비스 재실행
 * --> 우편번호 검색 창을 닫을 경우 element id 안에 우편번호 검색 html 이 삭제되는데,
 *     삭제 후 우편번호 서비스를 재실행한다.
 */
let useCheckStyle = true;

/**
 * useCheckStyle = true 일 경우
 * setTimeout() 함수에 의해 checkVisibilityChanged() 함수가 호출되는 주기
 * 기본: 250ms
 */
const interval = 250;

/**
 * useCheckStyle = true 일 경우
 * 우편번호 검색 이전 표시상태(true or false)
 */
let prevVisible = null;

/**
 * 우편번호 찾기화면을 iframe으로 삽입할 element id(필수)
 */
export let id = 'embeded_kakao_postcode';

/**
 * embeded element 의 스타일
 * 예) style="border:none; width:100%; height:100%; max-width:100%; max-height:100%;"
 *     style="border:solid #ddd #666;"
 */
export let style = '';

/**
 * 우편번호 검색창의 닫기 버튼 이미지 URL
 */
export let imgClose = '//t1.daumcdn.net/postcode/resource/images/close.png';

/**
 * 우편번호 검색창의 닫기 버튼 스타일
 */
export let imgCloseStyle = '';

/**
 * 우편번호 검색창의 닫기 버튼을 표시하지 않을 경우 사용
 * 예) <KakaoPostCode id="search_postcode" on:select={choosePostCode} hideCloseBtn></KakaoPostCode>
 */
export let hideCloseBtn = '';


/**
 * 카카오 우편번호 스크립트 로드 완료 시
 */
function onLoadScript() {
	isLoaded = true;

	if (useCheckStyle) {
		checkVisibilityChanged();
	} else {
		initialize();
	}
}

/**
 * 우편번호 검색 창 닫기
 */
function onClose() {
	// 우편번호 검색창 표시여부 초기화
	document.getElementById(id).style.display = 'none';
	prevVisible = false;

	// 우편번호 검색 html 삭제
	if (useCheckStyle && document.querySelector('#'+id+' div')) {
		document.querySelector('#'+id+' div').remove();
	}

	if (!useCheckStyle) {
		initialize();
	}
}

/**
 * 지정 시간 간격(interval)마다 우편번호 검색 엘리먼트의 표시여부 체크
 * 체크 CSS 속성: display, visibility
 */
function checkVisibilityChanged() {
	try {
		if (!document.getElementById(id)) {
			return;
		}
	} catch(e) {}

	let display = document.getElementById(id).style.display;
	if (display != undefined) {
		let curVisible = !(display.toLowerCase() == 'none');
		if (prevVisible != curVisible && curVisible == true) {
			prevVisible = curVisible;
			initialize();
		}
	}

	setTimeout(checkVisibilityChanged, interval);
}

/**
 * 주소 선택 시
 */
function onSelect(data) {
	let fullAddr = '';
	let extraAddr = '';

	// 도로명 주소 선택 시
	if (data.userSelectedType === 'R') {
		fullAddr = data.roadAddress;

		// 법정동명이 있을 경우
		if (data.bname !== '') {
			extraAddr += data.bname;
		}
		// 건물명이 있을 경우
		if (data.buildingName !== '') {
			extraAddr += (extraAddr !== '' ? ', ' + data.buildingName : data.buildingName);
		}
		fullAddr += (extraAddr !== '' ? ' ('+ extraAddr +')' : '');

	// 지번주소 선택 시
	} else {
		fullAddr = data.jibunAddress;
	}

	// select 이벤트 발생
	dispatch('select', {
		postcode: data.zonecode,
		address : fullAddr
	});
}

/**
 * 초기화
 */
function initialize() {
	if (!isLoaded) {
		return;
	}

	if (id == undefined || id == null || id == '') {
		id = 'embeded_kakao_postcode';
	}
	if (style == undefined) {
		style = '';
	}
	if (imgClose == undefined || imgClose == '') {
		imgClose = '//t1.daumcdn.net/postcode/resource/images/close.png';
	}
	if (imgCloseStyle == undefined) {
		imgCloseStyle = '';
	}
	if (hideCloseBtn == undefined) {
		hideCloseBtn = false;
	}

	let elementLayer = document.getElementById(id);

	// 우편번호 검색 실행
	window.daum.postcode.load(function() {
		new daum.Postcode({
			oncomplete: onSelect,
			width : '100%',
			height: '100%',
			maxSuggestItems: 5,
			onclose: function(state) {
				onClose();
			}
		}).embed(elementLayer);
	});
}
</script>
<style>
.kakao-postcode {
	position: absolute;
	left: 50%;
	top: 50%;
	width: 80vw;
	height: 80vh;
	max-width: 420px;
	max-height: 600px;
	overflow: auto;
	transform: translate(-50%, -50%);
	padding: 0;
	border:3px solid #000;
	border-radius: 0.2em;
	background: white;
	display: none;
}
.kakao-postcode-close {
	cursor: pointer;
	position: absolute;
	right: -2px;
	top: -2px;
	z-index: 1;
}
@media all and (max-width:480px) {
	.kakao-postcode {
		width: 90vw;
	}
}
</style>
<svelte:head>
{#if document.location.protocol == 'https:'}
	<script src="https://t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js?autoload=false" on:load={onLoadScript}></script>
{:else}
	<script src="http://dmaps.daum.net/map_js_init/postcode.v2.js?autoload=false" on:load={onLoadScript}></script>
{/if}
</svelte:head>

<div class="kakao-postcode" id={id} style={style}>
{#if hideCloseBtn != true}
	<img class="kakao-postcode-close" src={imgClose} style={imgCloseStyle} on:click={onClose} alt="" />
{/if}
</div>
