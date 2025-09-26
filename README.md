<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>상담일지 관리</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<style>
body { font-family: Arial, sans-serif; margin:0; padding:20px; background:#f4f6f9; }
h1 { text-align:center; color:#333; }
form { background:#fff; padding:15px; margin-bottom:20px; border-radius:8px; box-shadow:0 2px 6px rgba(0,0,0,0.1);}
label { display:block; margin:8px 0 4px; font-weight:bold; }
input, textarea, select, button { width:100%; padding:10px; margin-bottom:12px; border:1px solid #ccc; border-radius:6px; font-size:14px; box-sizing:border-box; }
textarea { resize:vertical; min-height:80px; max-height:300px; }
button { background:#007bff; color:#fff; cursor:pointer; font-weight:bold; }
button:hover { background:#0056b3; }
.list { list-style:none; padding:0; }
.list li { background:#fff; margin-bottom:10px; padding:15px; border-radius:8px; box-shadow:0 1px 4px rgba(0,0,0,0.1);}
.list h3 { margin:0 0 8px; color:#007bff; cursor:pointer; display:flex; justify-content:space-between; align-items:center;}
.actions button { width:auto; margin-left:5px; padding:5px 10px; font-size:13px; }
.entry-content { display:none; }
.entry-content img { max-width:100%; height:auto; margin-top:8px; border-radius:4px; }
@media(max-width:600px){ body{padding:10px;} form, .list li{padding:12px;} }
</style>
</head>
<body>
<h1>상담일지 관리</h1>

<form id="logForm">
<label for="site">설치 위치</label>
<input type="text" id="site" required>

<label for="address">설치 주소</label>
<input type="text" id="address" required>

<label for="phone">연락처</label>
<input type="text" id="phone" required>

<label for="content">상담 내용</label>
<textarea id="content" required></textarea>

<label for="fontSize">글자 크기</label>
<select id="fontSize">
<option value="14px">14px</option>
<option value="16px" selected>16px</option>
<option value="18px">18px</option>
<option value="20px">20px</option>
</select>

<label for="photos">사진 첨부</label>
<input type="file" id="photos" multiple accept="image/*">

<button type="submit">저장</button>
<button type="button" id="pdfBtn">PDF로 저장</button>
</form>

<ul class="list" id="logList"></ul>

<script>
const form=document.getElementById('logForm');
const siteInput=document.getElementById('site');
const addressInput=document.getElementById('address');
const phoneInput=document.getElementById('phone');
const contentInput=document.getElementById('content');
const fontSizeInput=document.getElementById('fontSize');
const photosInput=document.getElementById('photos');
const logList=document.getElementById('logList');
const pdfBtn=document.getElementById('pdfBtn');

let logs=JSON.parse(localStorage.getItem('logs'))||[];
let editIndex=null;

// 자동 저장 (5초마다)
setInterval(()=>{
    localStorage.setItem('draft', JSON.stringify({
        site: siteInput.value,
        address: addressInput.value,
        phone: phoneInput.value,
        content: contentInput.value,
        fontSize: fontSizeInput.value
    }));
},5000);

// 임시 저장 불러오기
const draft=JSON.parse(localStorage.getItem('draft'));
if(draft){
    siteInput.value=draft.site||'';
    addressInput.value=draft.address||'';
    phoneInput.value=draft.phone||'';
    contentInput.value=draft.content||'';
    fontSizeInput.value=draft.fontSize||'16px';
}

// 화면 렌더
function renderLogs(){
    logList.innerHTML='';
    logs.forEach((log,index)=>{
        const li=document.createElement('li');
        li.innerHTML=`
        <h3>
            ${log.site} (${log.address})
            <span>
                <button onclick="editLog(${index})">수정</button>
                <button onclick="deleteLog(${index})">삭제</button>
            </span>
        </h3>
        <div class="entry-content" style="font-size:${log.fontSize}">
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
    siteInput.value=log.site;
    addressInput.value=log.address;
    phoneInput.value=log.phone;
    contentInput.value=log.content;
    fontSizeInput.value=log.fontSize;
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
    const site=siteInput.value.trim();
    const address=addressInput.value.trim();
    const phone=phoneInput.value.trim();
    const content=contentInput.value.trim();
    const fontSize=fontSizeInput.value;
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
        const log={site,address,phone,content,fontSize,photos};
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
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    let y=10;
    const pageHeight = doc.internal.pageSize.getHeight();

    function addImageToDoc(imgSrc, callback){
        const img=new Image();
        img.src=imgSrc;
        img.onload=function(){
            let w=doc.internal.pageSize.getWidth()-20;
            let h=(img.height/img.width)*w;
            if(y+h>pageHeight){ doc.addPage(); y=10; }
            doc.addImage(img,'JPEG',10,y,w,h);
            y+=h+10;
            callback();
        }
    }

    function processLogs(i){
        if(i>=logs.length){
            doc.save('상담일지.pdf');
            return;
        }
        const log=logs[i];
        doc.setFontSize(16);
        doc.text(`설치 위치: ${log.site}`,10,y); y+=8;
        doc.setFontSize(14);
        doc.text(`설치 주소: ${log.address}`,10,y); y+=8;
        doc.text(`연락처: ${log.phone}`,10,y); y+=8;
        doc.text(`상담 내용: ${log.content}`,10,y); y+=10;

        let photoIndex=0;
        function addNextPhoto(){
            if(photoIndex>=log.photos.length){
                y+=10;
                processLogs(i+1);
                return;
            }
            addImageToDoc(log.photos[photoIndex], ()=>{
                photoIndex++;
                addNextPhoto();
            });
        }
        addNextPhoto();
    }

    processLogs(0);
});

// 초기 렌더
renderLogs();
</script>
</body>
</html>
