<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>상담일지</title>
<style>
body {
    font-family: Arial, sans-serif;
    margin: 20px;
    background-color: #fff;
    color: #000;
}
h1 {
    text-align: center;
}
form {
    display: flex;
    flex-direction: column;
    max-width: 800px;
    margin: 0 auto;
}
label {
    margin-top: 15px;
    font-weight: bold;
}
input, select, textarea {
    padding: 10px;
    font-size: 16px;
    margin-top: 5px;
    width: 100%;
    box-sizing: border-box;
    border: 1px solid #ccc;
    border-radius: 4px;
}
textarea {
    resize: vertical;
    min-height: 100px;
    max-height: 500px;
}
input[type="file"] {
    padding: 3px;
}
button {
    margin-top: 20px;
    padding: 12px;
    font-size: 18px;
    background-color: #007BFF;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}
button:hover {
    background-color: #0056b3;
}
#entries {
    max-width: 800px;
    margin: 20px auto 0;
}
.entry {
    border: 1px solid #ccc;
    border-radius: 4px;
    margin-top: 10px;
}
.entry-title {
    font-weight: bold;
    cursor: pointer;
    background-color: #f2f2f2;
    padding: 10px;
    border-radius: 4px;
}
.entry-content {
    display: none;
    padding: 10px;
}
.entry-content img {
    max-width: 100%;
    height: auto;
    margin-top: 10px;
}
@media (max-width: 600px) {
    body { margin: 10px; }
    input, select, textarea, button { font-size: 14px; }
}
</style>
</head>
<body>
<h1>상담일지</h1>
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

    <button type="button" onclick="saveEntry()">저장</button>
</form>

<h2>저장된 상담일지</h2>
<div id="entries"></div>

<script>
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

    if (!date || !name || !phone) {
        alert('상담일시, 성함, 전화번호는 필수입니다.');
        return;
    }

    const entryDiv = document.createElement('div');
    entryDiv.className = 'entry';

    const titleDiv = document.createElement('div');
    titleDiv.className = 'entry-title';
    titleDiv.textContent = address;
    entryDiv.appendChild(titleDiv);

    const contentDiv = document.createElement('div');
    contentDiv.className = 'entry-content';
    contentDiv.innerHTML = `
        <strong>상담일시:</strong> ${date}<br>
        <strong>성함:</strong> ${name}<br>
        <strong>전화번호:</strong> ${phone}<br>
        <strong>설치장소:</strong> ${place}<br>
        <strong>설치용량:</strong> ${capacity}<br>
        <strong>설치면적:</strong> ${area}<br>
        <strong>계약전력:</strong> ${contract}<br>
        <strong>세부 내용:</strong> ${details}<br>
    `;
    entryDiv.appendChild(contentDiv);

    // 사진 첨부
    if (photosInput.files.length > 0) {
        for (let i = 0; i < photosInput.files.length; i++) {
            const file = photosInput.files[i];
            const reader = new FileReader();
            reader.onload = function(e) {
                const img = document.createElement('img');
                img.src = e.target.result;
                contentDiv.appendChild(img);
            };
            reader.readAsDataURL(file);
        }
    }

    // 클릭하면 펼치기/접기
    titleDiv.addEventListener('click', () => {
        if (contentDiv.style.display === 'none' || contentDiv.style.display === '') {
            contentDiv.style.display = 'block';
        } else {
            contentDiv.style.display = 'none';
        }
    });

    document.getElementById('entries').prepend(entryDiv);
    document.getElementById('consultForm').reset();
}
</script>
</body>
</html>
