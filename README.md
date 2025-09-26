<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>상담일지</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&display=swap');

body { font-family: 'Noto Sans KR', sans-serif; margin:0; padding:20px; background:#f9f9f9;}
h1{text-align:center;}
label{display:block;margin-top:10px;}
input,textarea{width:100%;padding:8px;margin-top:4px;box-sizing:border-box;font-size:15px;}
textarea{min-height:100px;}
button{margin-top:15px;padding:10px 20px;font-size:15px;}
#records{margin-top:30px;}
.record{border:1px solid #ccc;padding:10px;margin-bottom:10px;background:#fff;}
.record img{max-width:100%;height:auto;margin-top:10px;}
</style>
</head>
<body>

<h1>상담일지</h1>

<form id="form">
  <label>상담일시<input type="datetime-local" id="dateTime" required></label>
  <label>전화번호<input type="text" id="phone" required></label>
  <label>설치주소<input type="text" id="address" required></label>
  <label>설치장소<input type="text" id="place" required></label>
  <label>설치용량<input type="text" id="capacity" required></label>
  <label>설치면적<input type="text" id="area" required></label>
  <label>계약전력<input type="text" id="power" required></label>
  <label>세부 내용<textarea id="details" required></textarea></label>
  <label>사진 첨부<input type="file" id="photo" accept="image/*" multiple></label>
  <button type="submit">추가/수정</button>
  <button type="button" id="download">PDF 생성</button>
  <button type="button" id="email">이메일 전송</button>
</form>

<div id="records"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
<script>
let records = JSON.parse(localStorage.getItem('records') || '[]');
const form = document.getElementById('form');
const recordsDiv = document.getElementById('records');
let editIndex = -1;

function saveRecords() { localStorage.setItem('records', JSON.stringify(records)); }

function renderRecords() {
  recordsDiv.innerHTML = '';
  records.forEach((r, idx) => {
    const div = document.createElement('div');
    div.className = 'record';
    div.innerHTML = `
      <strong>${r.address}</strong> (${r.dateTime})<br>
      전화번호: ${r.phone}<br>
      설치장소: ${r.place}<br>
      설치용량: ${r.capacity}<br>
      설치면적: ${r.area}<br>
      계약전력: ${r.power}<br>
      세부 내용: ${r.details}<br>
      ${r.photos.map(p=>`<img src="${p}">`).join('')}
      <button onclick="editRecord(${idx})">수정</button>
      <button onclick="deleteRecord(${idx})">삭제</button>
    `;
    recordsDiv.appendChild(div);
  });
}

function editRecord(idx){
  editIndex = idx;
  const r = records[idx];
  document.getElementById('dateTime').value = r.dateTime;
  document.getElementById('phone').value = r.phone;
  document.getElementById('address').value = r.address;
  document.getElementById('place').value = r.place;
  document.getElementById('capacity').value = r.capacity;
  document.getElementById('area').value = r.area;
  document.getElementById('power').value = r.power;
  document.getElementById('details').value = r.details;
}

function deleteRecord(idx){
  records.splice(idx,1);
  saveRecords();
  renderRecords();
}

form.addEventListener('submit', async (e) => {
  e.preventDefault();
  const files = Array.from(document.getElementById('photo').files);
  const photos = await Promise.all(files.map(file => new Promise(res=>{
    const reader = new FileReader();
    reader.onload = ()=>res(reader.result);
    reader.readAsDataURL(file);
  })));

  const record = {
    dateTime: document.getElementById('dateTime').value,
    phone: document.getElementById('phone').value,
    address: document.getElementById('address').value,
    place: document.getElementById('place').value,
    capacity: document.getElementById('capacity').value,
    area: document.getElementById('area').value,
    power: document.getElementById('power').value,
    details: document.getElementById('details').value,
    photos: photos
  };

  if(editIndex>=0){
    records[editIndex] = record;
    editIndex = -1;
  } else records.push(record);

  saveRecords();
  renderRecords();
  form.reset();
});

function generatePDF(callback){
  if(records.length===0){ alert('기록이 없습니다.'); return; }
  const element = document.createElement('div');
  element.style.fontFamily="'Noto Sans KR', sans-serif";
  element.style.fontSize='15px';
  records.forEach(r=>{
    const div=document.createElement('div');
    div.style.border='1px solid #ccc';
    div.style.padding='10px';
    div.style.marginBottom='10px';
    div.style.background='#fff';
    div.innerHTML=`
      <strong>${r.address}</strong> (${r.dateTime})<br>
      전화번호: ${r.phone}<br>
      설치장소: ${r.place}<br>
      설치용량: ${r.capacity}<br>
      설치면적: ${r.area}<br>
      계약전력: ${r.power}<br>
      세부 내용: ${r.details}<br>
    `;
    r.photos.forEach(p=>{
      const img=document.createElement('img');
      img.src=p;
      img.style.maxWidth='180mm';
      img.style.height='auto';
      img.style.marginTop='5px';
      div.appendChild(img);
    });
    element.appendChild(div);
  });
  const opt={margin:10, filename:'상담일지.pdf', image:{type:'jpeg',quality:0.98}, html2canvas:{scale:2,useCORS:true}, jsPDF:{unit:'mm',format:'a4',orientation:'portrait'}};
  html2pdf().set(opt).from(element).save().then(()=>{ if(callback) callback(); });
}

// PDF 생성 버튼
document.getElementById('download').addEventListener('click',()=>generatePDF());

// 이메일 전송 버튼
document.getElementById('email').addEventListener('click',()=>{
  generatePDF(()=>{
    // PDF 다운로드 안내
    alert("PDF가 생성되었습니다. PDF를 첨부하여 'dh_com@naver.com'으로 이메일을 보내주세요.");
    // mailto 링크 생성
    const subject = encodeURIComponent("상담일지 전송");
    const body = encodeURIComponent("안녕하세요,\n첨부된 상담일지를 확인해주세요.");
    window.location.href = `mailto:dh_com@naver.com?subject=${subject}&body=${body}`;
  });
});

renderRecords();
</script>

</body>
</html>
