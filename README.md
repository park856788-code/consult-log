<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>상담일지 앱</title>

<!-- PWA -->
<link rel="manifest" href="data:application/manifest+json,{}">
<meta name="theme-color" content="#007BFF">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-title" content="상담일지">
<link rel="icon" href="data:image/png;base64,iVBORw0KGgo=">

<style>
body { font-family: Arial, sans-serif; margin:0; padding:20px; background:#fff; color:#000; }
h1 { text-align:center; margin-bottom:20px;}
form { display:flex; flex-direction:column; max-width:800px; margin:0 auto; }
label { margin-top:15px; font-weight:bold; }
input, select, textarea { padding:10px; font-size:16px; margin-top:5px; width:100%; box-sizing:border-box; border:1px solid #ccc; border-radius:4px; }
textarea { resize:vertical; min-height:100px; max-height:500px; }
input[type="file"] { padding:3px; }
button { margin-top:10px; padding:12px; font-size:16px; background-color:#007BFF; color:#fff; border:none; border-radius:4px; cursor:pointer; }
button:hover { background-color:#0056b3; }
#entries { max-width:800px; margin:20px auto 0; display:flex; flex-direction:column; gap:10px; }
.entry { border:1px solid #ccc; border-radius:4px; overflow:hidden; }
.entry-title { font-weight:bold; cursor:pointer; background:#f2f2f2; padding:12px; border-bottom:1px solid #ccc; }
.entry-content { display:none; padding:12px; }
.entry-content img { max-width:100%; height:auto; margin-top:10px; border-radius:4px;}
@media (max-width:600px) { body{padding:10px;} input, select, textarea, button{font-size:14px;} }
</style>
</head>
<body>
<h1>상담일지 앱</h1>

<form id="consultForm">
<label for="date">상담일시</label>
<input type="datetime-local" id="date" required>
<label for="name">성함</label>
<input type="text" id="name" placeholder="이름 입력" required>
<label for="phone">전화번호</label>
<input type="tel" id="phone" placeholder="010-xxxx-xxxx" required>
<label for="address">설치주소</label>
<input type="text" id="address" placeholder="설치주소 입력" required>
<label for="place">설치장소</label>
<input type="text" id="place" placeholder="설치장소 입력" required>
<label for="capacity">설치용량</label>
<input type="text" id="capacity" placeholder="설치용량 입력" required>
<label for="area">설치면적</label>
<input type="text" id="area" placeholder="설치면적 입력" required>
<label for="contract">계약전력</label>
<input type="text" id="contract" placeholder="계약전력 입력" required>
<label for="details">세부 내용</label>
<textarea id="details" placeholder="세부 내용 입력" required></textarea>
<label for="photos">사진 첨부</label>
<input type="file" id="photos" multiple accept="image/*">

<div style="display:flex; flex-wrap:wrap; gap:10px; margin-top:10px;">
<button type="button" onclick="saveEntry()">저장</button>
<button type="button" onclick="downloadEntries()">다운로드</button>
<button type="button" onclick="uploadEntries()">불러오기</button>
<input type="file" id="uploadFile" style="display:none;" accept=".json">
</div>
</form>

<h2>저장된 상담일지</h2>
<div id="entries"></div>

<script>
let allEntries = [];

// 저장
function saveEntry() {
    const date = document.getElementById('date').value;
    const name = document.getElementById('name').value;
    const phone = document.getElementById('phone').value;
    const address = document.getElementById('address').value;
    const place = document.getElementById('place').value;
    const capacity = document.getElementById('capacity').value;
    const area = document.getElementById('area').value;
    const contract = document.getElementById('contract').value;
    const details = document.getElementById('details').value;
    const photosInput = document.getElementById('photos');

    if(!date||!name||!phone||!address){
        alert('상담일시, 성함, 전화번호, 설치주소는 필수입니다.');
        return;
    }

    const photos = [];
    if(photosInput.files.length>0){
        for(let i=0;i<photosInput.files.length;i++){
            const reader = new FileReader();
            reader.onload = function(e){ photos.push(e.target.result); };
            reader.readAsDataURL(photosInput.files[i]);
        }
    }

    const entry = {date,name,phone,address,place,capacity,area,contract,details,photos};
    allEntries.unshift(entry);
    renderEntries();
    document.getElementById('consultForm').reset();
}

// 렌더링
function renderEntries(){
    const container = document.getElementById('entries');
    container.innerHTML='';
    allEntries.forEach((entry)=>{
        const div = document.createElement('div');
        div.className='entry';
        const title = document.createElement('div');
        title.className='entry-title';
        title.textContent=entry.address;
        const content = document.createElement('div');
        content.className='entry-content';
        content.innerHTML=`
            <strong>상담일시:</strong> ${entry.date}<br>
            <strong>성함:</strong> ${entry.name}<br>
            <strong>전화번호:</strong> ${entry.phone}<br>
            <strong>설치장소:</strong> ${entry.place}<br>
            <strong>설치용량:</strong> ${entry.capacity}<br>
            <strong>설치면적:</strong> ${entry.area}<br>
            <strong>계약전력:</strong> ${entry.contract}<br>
            <strong>세부 내용:</strong> ${entry.details}<br>
        `;
        if(entry.photos){
            entry.photos.forEach(src=>{
                const img=document.createElement('img'); img.src=src; content.appendChild(img);
            });
        }
        title.addEventListener('click',()=>{ content.style.display=(content.style.display==='block')?'none':'block'; });
        div.appendChild(title); div.appendChild(content);
        container.appendChild(div);
    });
}

// 다운로드
function downloadEntries(){
    const blob = new Blob([JSON.stringify(allEntries)],{type:'application/json'});
    const a=document.createElement('a');
    a.href=URL.createObjectURL(blob);
    a.download='consult_entries.json';
    a.click();
}

// 업로드
function uploadEntries(){
    document.getElementById('uploadFile').click();
}
document.getElementById('uploadFile').addEventListener('change',function(e){
    const file=e.target.files[0];
    if(!file) return;
    const reader = new FileReader();
    reader.onload = function(ev){
        try{
            allEntries = JSON.parse(ev.target.result);
            renderEntries();
        }catch(err){ alert('파일 형식이 올바르지 않습니다.'); }
    };
    reader.readAsText(file);
});

// 오프라인 PWA 지원
if('serviceWorker' in navigator){
    const swCode=`const CACHE_NAME='consult-cache-v1';const urlsToCache=['./'];self.addEventListener('install',e=>{e.waitUntil(caches.open(CACHE_NAME).then(c=>c.addAll(urlsToCache)));});self.addEventListener('fetch',e=>{e.respondWith(caches.match(e.request).then(r=>r||fetch(e.request)));});`;
    const blob = new Blob([swCode],{type:'application/javascript'});
    const swUrl = URL.createObjectURL(blob);
    navigator.serviceWorker.register(swUrl)
    .then(reg=>console.log('ServiceWorker 등록됨',reg))
    .catch(err=>console.log('ServiceWorker 등록 실패',err));
}
</script>
</body>
</html>
