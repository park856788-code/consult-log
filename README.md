# consult-log
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>상담 일지 (사진 첨부)</title>
<style>
body {
  font-family: 'Noto Sans KR', sans-serif;
  margin:0; padding:0;
  background:#f5f6fa;
  display:flex; justify-content:center;
}
.container {
  width:95%; max-width:700px;
  background:#fff;
  margin:20px; padding:20px;
  border-radius:12px;
  box-shadow:0 4px 10px rgba(0,0,0,0.1);
}
h1{text-align:center; margin-bottom:20px; color:#333;}
label{
  font-weight:bold; display:block; margin:10px 0 5px; color:#444;
}
input,textarea,select{
  width:100%; padding:12px; border:1px solid #ccc; border-radius:6px; font-size:16px; box-sizing:border-box;
}
textarea{min-height:120px; resize:vertical;}
.controls{display:flex; justify-content:space-between; margin-top:10px;}
.controls label{font-weight:normal;}
button{
  width:100%; padding:15px; margin-top:15px; border:none; background:#4CAF50; color:white; font-size:18px; border-radius:8px; cursor:pointer;
}
button:hover{background:#45a049;}
.list{margin-top:30px;}
.entry{
  padding:15px; border:1px solid #ddd; border-radius:8px; margin-bottom:10px; background:#fafafa;
}
.entry h3{margin:0 0 10px; font-size:18px; color:#333;}
.entry p{margin:4px 0; font-size:14px; color:#555;}
.entry img{max-width:100%; margin-top:8px; border-radius:6px;}
.entry button{
  background:#f44336; font-size:14px; padding:8px 12px; width:auto; margin-top:8px; border:none; border-radius:6px; cursor:pointer;
}
.entry button:hover{background:#d32f2f;}
@media (max-width:480px){
  .container{margin:10px; padding:15px;}
  input,textarea{font-size:14px;}
  button{font-size:16px; padding:12px;}
}
</style>
</head>
<body>
<div class="container">
<h1>📋 상담 일지 (사진 첨부)</h1>
<form id="logForm">
  <label>상담일시</label>
  <input type="datetime-local" name="상담일시">

  <label>전화번호</label>
  <input type="tel" name="전화번호" placeholder="010-1234-5678">

  <label>설치주소</label>
  <input type="text" name="설치주소">

  <label>설치장소</label>
  <input type="text" name="설치장소">

  <label>설치용량</label>
  <input type="text" name="설치용량">

  <label>설치면적</label>
  <input type="text" name="설치면적">

  <label>계약전력</label>
  <input type="text" name="계약전력">

  <label>세부 내용</label>
  <textarea id="details" name="세부내용"></textarea>

  <label>현장 사진 첨부</label>
  <input type="file" id="photo" accept="image/*">

  <div class="controls">
    <label>글자 크기
      <select id="fontSize">
        <option value="14px">작게</option>
        <option value="16px" selected>보통</option>
        <option value="20px">크게</option>
      </select>
    </label>
    <label>글꼴
      <select id="fontFamily">
        <option value="'Noto Sans KR', sans-serif">기본</option>
        <option value="'Gulim', sans-serif">굴림</option>
        <option value="'Batang', serif">바탕</option>
      </select>
    </label>
  </div>

  <button type="button" onclick="saveEntry()">💾 상담일지 저장</button>
</form>

<div class="list">
<h2>📑 저장된 상담일지</h2>
<div id="entries"></div>
</div>
</div>

<script>
// 글꼴 및 크기 조정
const details = document.getElementById("details");
document.getElementById("fontSize").addEventListener("change", e => details.style.fontSize=e.target.value);
document.getElementById("fontFamily").addEventListener("change", e => details.style.fontFamily=e.target.value);

// 상담일지 저장
function saveEntry(){
  const form=document.getElementById("logForm");
  const data=new FormData(form);
  let entry={};
  data.forEach((v,k)=>{entry[k]=v;});
  entry.id=Date.now();

  // 사진 처리
  const fileInput=document.getElementById("photo");
  if(fileInput.files && fileInput.files[0]){
    const reader=new FileReader();
    reader.onload=function(e){
      entry.photo=e.target.result;
      storeEntry(entry);
    }
    reader.readAsDataURL(fileInput.files[0]);
  } else {
    entry.photo=null;
    storeEntry(entry);
  }
}

// 저장 후 렌더링
function storeEntry(entry){
  let entries=JSON.parse(localStorage.getItem("entries"))||[];
  entries.push(entry);
  localStorage.setItem("entries",JSON.stringify(entries));
  document.getElementById("logForm").reset();
  renderEntries();
}

// 리스트 렌더링
function renderEntries(){
  const entriesDiv=document.getElementById("entries");
  entriesDiv.innerHTML="";
  let entries=JSON.parse(localStorage.getItem("entries"))||[];
  entries.reverse().forEach(entry=>{
    const div=document.createElement("div");
    div.className="entry";
    div.innerHTML=`
      <h3>${entry["상담일시"]||"미입력"}</h3>
      <p><b>전화번호:</b> ${entry["전화번호"]||""}</p>
      <p><b>설치주소:</b> ${entry["설치주소"]||""}</p>
      <p><b>설치장소:</b> ${entry["설치장소"]||""}</p>
      <p><b>설치용량:</b> ${entry["설치용량"]||""}</p>
      <p><b>설치면적:</b> ${entry["설치면적"]||""}</p>
      <p><b>계약전력:</b> ${entry["계약전력"]||""}</p>
      <p><b>세부내용:</b> ${entry["세부내용"]||""}</p>
      ${entry.photo ? `<img src="${entry.photo}" alt="현장사진">` : ""}
      <button onclick="deleteEntry(${entry.id})">🗑 삭제</button>
    `;
    entriesDiv.appendChild(div);
  });
}

// 삭제
function deleteEntry(id){
  let entries=JSON.parse(localStorage.getItem("entries"))||[];
  entries=entries.filter(e=>e.id!==id);
  localStorage.setItem("entries",JSON.stringify(entries));
  renderEntries();
}

// 초기 렌더링
renderEntries();

// 간단한 PWA 기능
if('serviceWorker' in navigator){
  navigator.serviceWorker.register(URL.createObjectURL(new Blob([`
    self.addEventListener('install',e=>{e.waitUntil(caches.open('cache-v1').then(c=>c.addAll([])))});
    self.addEventListener('fetch',e=>{e.respondWith(fetch(e.request).catch(()=>caches.match(e.request)))});
  `],{type:'text/javascript'})));
}
</script>
</body>
</html>
