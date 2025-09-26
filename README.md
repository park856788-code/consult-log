<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>상담일지 관리</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
<style>
body { font-family: 'Nanum Gothic', Arial, sans-serif; margin:0; padding:20px; background:#f4f6f9; }
h1 { text-align:center; color:#333; }
form { background:#fff; padding:15px; margin-bottom:20px; border-radius:8px; box-shadow:0 2px 6px rgba(0,0,0,0.1);}
label { display:block; margin:8px 0 4px; font-weight:bold; }
input, textarea, button { width:100%; padding:10px; margin-bottom:12px; border:1px solid #ccc; border-radius:6px; font-size:14px; box-sizing:border-box; }
textarea { resize:vertical; min-height:120px; max-height:400px; font-size:15px; }
button { background:#007bff; color:#fff; cursor:pointer; font-weight:bold; }
button:hover { background:#0056b3; }
.list { list-style:none; padding:0; }
.list li { background:#fff; margin-bottom:10px; padding:15px; border-radius:8px; box-shadow:0 1px 4px rgba(0,0,0,0.1);}
.list h3 { margin:0 0 8px; color:#007bff; cursor:pointer; display:flex; justify-content:space-between; align-items:center;}
.actions button { width:auto; margin-left:5px; padding:5px 10px; font-size:13px; }
.entry-content { display:none; font-family: 'Nanum Gothic', Arial, sans-serif; font-size:15px;}
.entry-content img { max-width:100%; height:auto; margin-top:8px; border-radius:4px; }
@media(max-width:600px){ body{padding:10px;} form, .list li{padding:12px;} }
</style>
</head>
<body>
<h1>상담일지 관리</h1>

<form id="logForm">
<label for="datetime">상담일시</label>
<input type="datetime-local" id="datetime">

<label for="site">설치 위치</label>
<input type="text" id="site" required>

<label for="address">설치 주소</label>
<input type="text" id="address" required>

<label for="phone">연락처</label>
<input type="text" id="phone" required>

<label for="content">상담 내용</label>
<textarea id="content" required></textarea>

<label for="photos">사진 첨부</label>
<input type="file" id="photos" multiple accept="image/*">

<button type="submit">저장</button>
<button type="button" id="pdfBtn">PDF로 저장</button>
<button type="button" id="naverBtn">네이버 마이박스 열기</button>
</form>

<ul class="list" id="logList"></ul>

<script>
const form=document.getElementById('logForm');
const datetimeInput=document.getElementById('datetime');
const siteInput=document.getElementById('site');
const addressInput=document.getElementById('address');
const phoneInput=document.getElementById('phone');
const contentInput=document.getElementById('content');
const photosInput=document.getElementById('photos');
const logList=document.getElementById('logList');
const pdfBtn=document.getElementById('pdfBtn');
const naverBtn=document.getElementById('naverBtn');

let logs=JSON.parse(localStorage.getItem('logs'))||[];
let editIndex=null;

// 자동 저장 (5초마다)
setInterval(()=>{
    localStorage.setItem('draft', JSON.stringify({
        datetime: datetimeInput.value,
        site: siteInput.value,
        address: addressInput.value,
        phone: phoneInput.value,
        content: contentInput.value
    }));
},5000);

// 임시 저장 불러오기
const draft=JSON.parse(localStorage.getItem('draft'));
if(draft){
    datetimeInput.value=draft.datetime||'';
    siteInput.value=draft.site||'';
    addressInput.value=draft.address||'';
    phoneInput.value=draft.phone||'';
    contentInput.value=draft.content||'';
}

// 화면 렌더
function renderLogs(){
    logList.innerHTML='';
    logs.forEach((log,index)=>{
        const li=document.createElement('li');
        li.innerHTML=`
        <h3>
            ${log.site} (${log.address}) - ${log.datetime || ''}
            <span>
                <button onclick="editLog(${index})">수정</button>
                <button onclick="deleteLog(${index})">삭제</button>
            </span>
        </h3>
        <div class="entry-content">
            <p><strong>연락처:</strong> ${log.phone}</p>
            <p>${log.content}</p>
            ${log.photos.map(src=>`<img src="${src}">`).join('')}
        </div>`;
        const header=li.querySelector('h3');
        const contentDiv=li.querySelector('.entry-content');
        header.addEventListener('click',e=>{
            if(!e.target.closest('button')){
                contentDiv.style.display=contentDiv.style.display==='block'?'none':'block';
            }
        });
        logList.appendChild(li);
    });
}

// 수정
window.editLog=function(index){
    const log=logs[index];
    datetimeInput.value=log.datetime||'';
    siteInput.value=log.site;
    addressInput.value=log.address;
    phoneInput.value=log.phone;
    contentInput.value=log.content;
    editIndex=index;
};

// 삭제
window.deleteLog=function(index){
    if(confirm("정말 삭제하시겠습니까?")){
        logs.splice(index,1);
        localStorage.setItem('logs',JSON.stringify(logs));
        renderLogs();
    }
};

// 저장
form.addEventListener('submit',e=>{
    e.preventDefault();
    const datetime=datetimeInput.value || new Date().toISOString().slice(0,16);
    const site=siteInput.value.trim();
    const address=addressInput.value.trim();
    const phone=phoneInput.value.trim();
    const content=contentInput.value.trim();
    const files=Array.from(photosInput.files);

    if(!site||!address||!phone||!content) return alert('모든 필드를 입력해주세요.');

    const photos=[];
    if(files.length===0) finalizeEntry();
    else{
        let count=0;
        files.forEach(file=>{
            const reader=new FileReader();
            reader.onload=e=>{
                const img=new Image();
                img.src=e.target.result;
                img.onload=function(){
                    const canvas=document.createElement('canvas');
                    const MAX_WIDTH=800;
                    const scale=Math.min(MAX_WIDTH/img.width,1);
                    canvas.width=img.width*scale;
                    canvas.height=img.height*scale;
                    const ctx=canvas.getContext('2d');
                    ctx.drawImage(img,0,0,canvas.width,canvas.height);
                    photos.push(canvas.toDataURL('image/jpeg',0.7));
                    count++;
                    if(count===files.length) finalizeEntry();
                }
            };
            reader.readAsDataURL(file);
        });
    }

    function finalizeEntry(){
        const log={datetime,site,address,phone,content,fontSize:'15px',photos};
        if(editIndex===null) logs.unshift(log);
        else{ logs[editIndex]=log; editIndex=null; }
        localStorage.setItem('logs',JSON.stringify(logs));
        localStorage.removeItem('draft'); // 임시 저장 삭제
        renderLogs();
        form.reset();
    }
});

// PDF 생성
pdfBtn.addEventListener('click',()=>{
    if(logs.length===0)return alert("저장된 상담일지가 없습니다.");
    const element = document.createElement('div');
    logs.forEach(log=>{
        const div=document.createElement('div');
        div.style.marginBottom='15px';
        div.innerHTML=`
        <p><strong>상담일시:</strong> ${log.datetime}</p>
        <p><strong>설치 위치:</strong> ${log.site}</p>
        <p><strong>설치 주소:</strong> ${log.address}</p>
        <p><strong>연락처:</strong> ${log.phone}</p>
        <p>${log.content}</p>
        ${log.photos.map(src=>`<img src="${src}" style="max-width:500px;margin-top:5px;">`).join('')}
        <hr>`;
        element.appendChild(div);
    });
    html2pdf().from(element).set({
        margin:10,
        filename:'상담일지.pdf',
        image:{type:'jpeg', quality:0.7},
        html2canvas:{scale:2, useCORS:true},
        jsPDF:{unit:'mm', format:'a4', orientation:'portrait'}
    }).save();
});

// 네이버 마이박스 열기
naverBtn.addEventListener('click',()=>{
    window.open('https://nid.naver.com/nidlogin.login', '_blank');
});

// 초기 렌더
renderLogs();
</script>
</body>
</html>
