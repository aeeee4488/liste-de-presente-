# liste-de-presente-
1Bacsief math
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>حضور 1Bac Sef - رياضيات</title>
<script defer src="https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js"></script>
<style>
body{font-family:Arial;background:#0f172a;color:white;text-align:center;padding:15px}
.card{background:#1e293b;padding:15px;border-radius:15px;margin:10px 0}
button{padding:12px 20px;border:none;border-radius:10px;background:#22c55e;color:black;font-weight:bold;width:90%;margin:5px;font-size:16px}
input{padding:10px;border-radius:8px;width:80%;margin:5px}
video{width:100%;border-radius:15px;border:2px solid #22c55e}
.present{color:#22c55e} .absent{color:#ef4444}
</style>
</head>
<body>
<h2>📚 حضور مادة الرياضيات - 1Bac Sef</h2>
<div class="card">
<h3>1- تسجيل تلميذ جديد</h3>
<input id="nameInput" placeholder="اسم التلميذ مثلا: احمد">
<button onclick="startEnroll()">فتح الكاميرا</button>
<video id="video" autoplay muted playsinline></video>
<button onclick="saveFace()">حفظ وجه التلميذ</button>
<p id="status"></p>
</div>

<div class="card">
<h3>2- بدء حصة اليوم</h3>
<button onclick="startAttendance()" style="background:#3b82f6;color:white">📷 بدء تسجيل الحضور</button>
<div id="result"></div>
<button onclick="exportExcel()" style="background:white">📥 تحميل Excel</button>
</div>

<script>
let labeledDescriptors = JSON.parse(localStorage.getItem('faces') || '[]');
let faceMatcher;
let stream;
// تحميل الموديل
Promise.all([
  faceapi.nets.tinyFaceDetector.loadFromUri('https://justadudewhohacks.github.io/face-api.js/models'),
  faceapi.nets.faceLandmark68Net.loadFromUri('https://justadudewhohacks.github.io/face-api.js/models'),
  faceapi.nets.faceRecognitionNet.loadFromUri('https://justadudewhohacks.github.io/face-api.js/models')
]).then(()=>{document.getElementById('status').innerText='✅ النظام واجد';});

async function startEnroll(){ stream = await navigator.mediaDevices.getUserMedia({video:{}}); document.getElementById('video').srcObject = stream; }
async function saveFace(){
  const name = document.getElementById('nameInput').value; if(!name) return alert('كتب السمية');
  const detection = await faceapi.detectSingleFace(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceDescriptor();
  if(!detection) return alert('ما باينش الوجه مزيان، قرب شوية للضو');
  let existing = labeledDescriptors.find(f=>f.label===name);
  if(existing){ existing.descriptors.push(Array.from(detection.descriptor)); } else { labeledDescriptors.push({label:name, descriptors:[Array.from(detection.descriptor)]}); }
  localStorage.setItem('faces', JSON.stringify(labeledDescriptors));
  document.getElementById('status').innerText = `تم حفظ ${name} بنجاح ✅ (${labeledDescriptors.length}/40)`;
  document.getElementById('nameInput').value='';
}
let presentSet = new Set();
async function startAttendance(){
  if(labeledDescriptors.length==0) return alert('سجل التلاميذ أولا');
  if(!stream){ stream = await navigator.mediaDevices.getUserMedia({video:{}}); document.getElementById('video').srcObject = stream; }
  const descriptors = labeledDescriptors.map(d=> new faceapi.LabeledFaceDescriptors(d.label, d.descriptors.map(x=> new Float32Array(x))));
  faceMatcher = new faceapi.FaceMatcher(descriptors, 0.55);
  setInterval(async ()=>{
    const detection = await faceapi.detectSingleFace(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceDescriptor();
    if(detection){ const best = faceMatcher.findBestMatch(detection.descriptor); if(best.label!=='unknown'){ presentSet.add(best.label); updateList(); } }
  }, 1500);
}
function updateList(){
  let html = '<h4>الحاضرون:</h4>';
  labeledDescriptors.forEach(s=>{
    if(presentSet.has(s.label)) html+= `<p class="present">✅ ${s.label} - حاضر</p>`;
    else html+= `<p class="absent">❌ ${s.label} - غائب</p>`;
  });
  document.getElementById('result').innerHTML = html + `<p>المجموع: ${presentSet.size} / ${labeledDescriptors.length}</p>`;
}
function exportExcel(){
  let csv = "الاسم,الحالة,التاريخ\n";
  const date = new Date().toLocaleDateString();
  labeledDescriptors.forEach(s=>{
    csv += `${s.label},${presentSet.has(s.label)?'حاضر':'غائب'},${date}\n`;
  });
  const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href=url; a.download=`ghiyab-1Bac-${date}.csv`; a.click();
}
</script>
</body>
</html>