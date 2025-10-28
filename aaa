<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <title>Mô phỏng phản xạ ánh sáng trên gương phẳng (p5.js)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <!-- p5.js -->
  <script src="https://cdn.jsdelivr.net/npm/p5@1.9.0/lib/p5.min.js"></script>
  <style>
    :root{--green:#28a745;--greenDark:#156f2f;--blue:#1877f2;--border:#333}
    body{margin:0;background:#fff;font-family:system-ui,Segoe UI,Roboto,Arial}
    .wrap{max-width:980px;margin:16px auto;padding:16px;border:2px solid var(--border);border-radius:12px}
    h1{font-size:20px;margin:0 0 10px}
    .row{display:flex;gap:16px;flex-wrap:wrap}
    .panel{flex:1 1 300px;border:1px solid #ddd;border-radius:12px;padding:12px}
    .ctrl{display:flex;align-items:center;gap:8px;margin:6px 0}
    .label{font-weight:600;margin-top:6px}
    canvas{display:block;border:1px solid #e6e6e6;border-radius:12px}
    button{padding:8px 12px;border-radius:8px;border:1px solid #222;background:#f5f5f5;cursor:pointer}
    button:hover{background:#eee}
    input[type="range"]{width:180px}
    .warn{font-weight:700;color:#c00;animation:blink 1s linear infinite}
    @keyframes blink{50%{opacity:.2}}
    .note{font-size:12px;color:#555}
  </style>
</head>
<body>
  <div class="wrap">
    <h1>Mô phỏng phản xạ ánh sáng trên gương phẳng</h1>
    <div class="row">
      <div class="panel">
        <div class="label">Bật/tắt</div>
        <div class="ctrl"><input id="chkImage" type="checkbox" checked><label for="chkImage">Ảnh ảo (S′)</label></div>
        <div class="ctrl"><input id="chkNormal" type="checkbox" checked><label for="chkNormal">Pháp tuyến (nét đứt)</label></div>
        <div class="ctrl"><input id="chkAngles" type="checkbox" checked><label for="chkAngles">Góc i, i′ (độ)</label></div>
        <div class="ctrl"><input id="chkVirtualRay" type="checkbox" checked><label for="chkVirtualRay">Tia ảo (nét đứt xanh)</label></div>

        <div class="label">Thanh trượt</div>
        <div class="ctrl"><label for="sObj">Chiều cao ngọn nến</label><input id="sObj" type="range" min="50" max="180" value="100"></div>
        <div class="ctrl"><label for="sMirror">Chiều cao gương</label><input id="sMirror" type="range" min="160" max="520" value="360"></div>

        <div class="ctrl"><button id="btnReset">Đặt lại</button></div>
        <p class="note">Đỏ: tia tới/ phản xạ (qua chân nến) • Xanh đứt: tia ảo • Pháp tuyến: nét đứt • Ảnh ảo: nến mờ.</p>
      </div>

      <div class="panel" style="flex:2 1 520px">
        <div id="canvas-holder"></div>
        <div id="warning" style="min-height:1.2em;margin-top:8px"></div>
      </div>
    </div>
  </div>

<script>
/* ===== KÍCH THƯỚC CANVAS ===== */
let W=920, H=520;

/* ===== TRẠNG THÁI ===== */
let xMirror=0, mirrorH=360;              // gương x=0
let xS=-260, yS=40, hS=100;              // tâm nến & chiều cao
let yBaseS=yS + hS/2;                    // CHÂN NẾN (cố định khi đổi chiều cao bằng slider)
let xEye=-320, yEye=-100;                // mắt

let dragging=null, grabDX=0, grabDY=0;

let showImage=true, showNormal=true, showAngles=true, showVirtualRay=true;

/* ===== DOM ===== */
let chkImage, chkNormal, chkAngles, chkVirtualRay, sObj, sMirror, btnReset, warnEl;

/* ===== TIỆN ÍCH ===== */
function toCanvas([x,y]){return [W/2+x, H/2+y]}
function fromCanvas(px,py){return [px-W/2, py-H/2]}
function clamp(v,a,b){return Math.max(a, Math.min(b,v))}
function lineIntersectX0(A,B){
  const [x1,y1]=A, [x2,y2]=B; const dx=x2-x1, dy=y2-y1;
  if (Math.abs(dx)<1e-9) return null;
  const t=-x1/dx; const y=y1+t*dy; return [0,y,t];
}
function dot(a,b){return a[0]*b[0]+a[1]*b[1]}
function len(v){return Math.hypot(v[0],v[1])}
function norm(v){const L=len(v); return L?[v[0]/L,v[1]/L]:[0,0]}
function angleBetween(u,v){const nu=norm(u), nv=norm(v); const c=Math.max(-1,Math.min(1,dot(nu,nv))); return Math.acos(c)}
function deg(rad){return rad*180/Math.PI}

/* ===== P5 ===== */
function setup(){
  const c=createCanvas(W,H); c.parent("canvas-holder");
  angleMode(RADIANS);

  chkImage = document.getElementById('chkImage');
  chkNormal= document.getElementById('chkNormal');
  chkAngles= document.getElementById('chkAngles');
  chkVirtualRay=document.getElementById('chkVirtualRay');
  sObj     = document.getElementById('sObj');
  sMirror  = document.getElementById('sMirror');
  btnReset = document.getElementById('btnReset');
  warnEl   = document.getElementById('warning');

  chkImage.onchange = ()=> showImage = chkImage.checked;
  chkNormal.onchange= ()=> showNormal= chkNormal.checked;
  chkAngles.onchange= ()=> showAngles= chkAngles.checked;
  chkVirtualRay.onchange= ()=> showVirtualRay=chkVirtualRay.checked;
  sMirror.oninput   = ()=> mirrorH  = +sMirror.value;

  // Chiều cao nến: giữ CHÂN cố định, chỉ đỉnh thay đổi
  sObj.oninput = ()=>{
    const newH=+sObj.value; hS=newH; yS = yBaseS - hS/2;
    // giữ trong khung
    const topY = yS - hS/2, minTop = -H/2 + 10;
    if(topY < minTop){ const d=minTop - topY; yS += d; yBaseS += d; }
  };

  btnReset.onclick = resetAll;
}

function resetAll(){
  mirrorH=360; sMirror.value=mirrorH;
  hS=100; sObj.value=hS;
  xS=-260; yS=40; yBaseS=yS + hS/2;
  xEye=-320; yEye=-100;
  showImage=showNormal=showAngles=showVirtualRay=true;
  chkImage.checked=chkNormal.checked=chkAngles.checked=chkVirtualRay.checked=true;
}

function draw(){
  background(255);
  noStroke(); fill(30); textSize(14); textAlign(LEFT,TOP);
  text("Đỏ: tia tới/ phản xạ qua chân nến • Xanh đứt: tia ảo • Pháp tuyến: nét đứt • Ảnh ảo: mờ",12,10);

  drawMirror(); // gương + gạch chéo 45°

  // Dựng tia qua CHÂN NẾN
  const S_base=[xS,yBaseS];           // chân nến
  const S_base_img=[-xS,yBaseS];      // chân ảnh ảo
  const Eye=[xEye,yEye];

  // I: giao (Eye -> chân ảnh ảo) với gương x=0
  const Ires=lineIntersectX0(Eye,S_base_img);
  const I = Ires ? [Ires[0], Ires[1]] : [0,0];

  // Tia tới & phản xạ (đỏ)
  stroke(220,0,0); strokeWeight(2);
  drawSegment(S_base, I);
  drawSegment(I, Eye);

  // Tia ảo (xanh dương, nét đứt)
  if (showVirtualRay){
    stroke(24,119,242); strokeWeight(2); drawingContext.setLineDash([6,6]);
    drawSegment(I, S_base_img);
    drawingContext.setLineDash([]);
  }

  // Pháp tuyến tại I (nét đứt xanh nhạt) — gương dọc ⇒ pháp tuyến ngang qua I (trong hệ y xuống)
  if (showNormal){
    stroke(0,160,220); strokeWeight(1.6); drawingContext.setLineDash([6,6]);
    drawSegment([-W, I[1]],[W, I[1]]);
    drawingContext.setLineDash([]);
    drawLabel("Pháp tuyến", [8, I[1]-18]);
  }

  // Vẽ nến & mắt
  drawCandle([xS,yS], hS, false);
  drawEyeProfile([xEye,yEye], "Mắt");

  // Ảnh ảo S′ (giống nến thật nhưng mờ)
  if (showImage){
    drawCandle([-xS,yS], hS, true);
    drawLabel("Ảnh ảo S′", [-xS+10, (yS - hS/2) - 20]);
  }

  // Điểm I & nhãn
  fill(0); noStroke(); drawPoint(I,5); drawLabel("I", [I[0]-14, I[1]-16]);
  drawLabel("Gương", [8, -mirrorH/2 - 24]);

  // Góc i, i′
  if (showAngles){
    const n=[-1,0]; // pháp tuyến hướng sang trái
    const vInc=[S_base[0]-I[0], S_base[1]-I[1]];
    const vRef=[Eye[0]-I[0],   Eye[1]-I[1]];
    const angI = deg(angleBetween(n, vInc));
    const angR = deg(angleBetween(n, vRef));
    drawAngleArc(I, n, vInc, 26);
    drawAngleArc(I, n, vRef, 38);
    fill(220,0,0); noStroke(); textSize(13);
    const t1=toCanvas([I[0]+8,I[1]-10]); text("i ≈ "+nf(angI,1,1)+"°", t1[0], t1[1]);
    const t2=toCanvas([I[0]+8,I[1]+4 ]); text("i′ ≈ "+nf(angR,1,1)+"°", t2[0], t2[1]);
  }

  // Cảnh báo: mắt có thấy toàn bộ ảnh?
  const topImg=[-xS, yS - hS/2], botImg=[-xS, yBaseS];
  const yI_top=lineIntersectX0(Eye, topImg)?.[1];
  const yI_bot=lineIntersectX0(Eye, botImg)?.[1];
  const halfH = mirrorH/2;
  const seeAll = (yI_top!==undefined && yI_bot!==undefined &&
                  yI_top<=halfH && yI_top>=-halfH &&
                  yI_bot<=halfH && yI_bot>=-halfH);
  warnEl.innerHTML = seeAll ? "" : '<span class="warn">⚠️ Vị trí đặt mắt này không thấy toàn bộ ảnh</span>';
}

/* ===== VẼ GƯƠNG (đường xanh đậm + gạch chéo 45° phía sau) ===== */
function drawMirror(){
  const x0 = xMirror, y1 = -mirrorH/2, y2 = mirrorH/2;
  // đường gương
  stroke(getColor('--green')); strokeWeight(6);
  const a=toCanvas([x0,y1]), b=toCanvas([x0,y2]); line(a[0],a[1],b[0],b[1]);
  // gạch chéo 45° sang phải
  stroke(getColor('--greenDark')); strokeWeight(3);
  const step=20, len=26, drop=10;
  for(let y=y1; y<=y2; y+=step){
    const p1=toCanvas([x0+2, y - drop/2]);
    const p2=toCanvas([x0+2 + len, y + drop/2]);
    line(p1[0],p1[1],p2[0],p2[1]);
  }
}
function getColor(v){return getComputedStyle(document.documentElement).getPropertyValue(v)}

/* ===== CÁC HÀM VẼ KHÁC ===== */
function drawSegment(A,B){const p1=toCanvas(A), p2=toCanvas(B); line(p1[0],p1[1],p2[0],p2[1]);}
function drawPoint(P,r){const p=toCanvas(P); circle(p[0],p[1],r*2);}
function drawLabel(txt,P){const p=toCanvas(P); fill(20); noStroke(); textSize(13); textAlign(LEFT,BOTTOM); text(txt,p[0],p[1]);}

function drawCandle(S,h,isImage=false){
  // đỉnh & chân (chân = yBaseS luôn đi cùng nến khi kéo)
  const top=[S[0], S[1]-h/2], bot=[S[0], yBaseS];
  const pT=toCanvas(top), pB=toCanvas(bot);
  const alpha = isImage ? 90 : 255;

  // trục nến
  stroke(0,0,0,alpha); strokeWeight(3); line(pT[0],pT[1],pB[0],pB[1]);
  // thân nến
  strokeWeight(8); stroke(255,165,0, Math.min(150,alpha)); line(pT[0],pT[1]+8, pB[0],pB[1]-2);
  noStroke(); fill(255,140,0,alpha); ellipse(pT[0],pT[1]+3,10,6);
  // ngọn lửa
  fill(255,200,0,alpha); ellipse(pT[0],pT[1]-8,12,18);
  fill(255,100,0,alpha); ellipse(pT[0],pT[1]-10,6,10);

  drawLabel(isImage?"S′":"Ngọn nến", [S[0]-24, top[1]-14]);
}

function drawEyeProfile(E,label){
  const p=toCanvas(E), w=34, h=16;
  noFill(); stroke(0); strokeWeight(2);
  arc(p[0],p[1], w, h, Math.PI, 0); // mí trên
  arc(p[0],p[1], w, h, 0, Math.PI); // mí dưới
  // đồng tử lệch về phía gương (bên phải)
  const pupilX = p[0] + w*0.22;
  fill(0); noStroke(); circle(pupilX, p[1], 6);
  fill(255); circle(pupilX-2, p[1]-2, 2.5); // highlight
  // khóe mắt trái nhọn
  stroke(0); strokeWeight(2);
  line(p[0]-w/2, p[1], p[0]-w/2+6, p[1]-2);
  drawLabel(label,[E[0]-10,E[1]-18]);
}

function drawAngleArc(O,a,b,R){
  const angA=Math.atan2(a[1],a[0]), angB=Math.atan2(b[1],b[0]);
  let d=angB-angA; while(d>Math.PI)d-=2*Math.PI; while(d<-Math.PI)d+=2*Math.PI;
  noFill(); stroke(220,0,0); strokeWeight(1.8);
  const c=toCanvas(O); arc(c[0],c[1], R*2, R*2, angA, angA+d);
}

/* ===== KÉO THẢ ===== */
function mousePressed(){
  const m=fromCanvas(mouseX,mouseY);
  const nearEye = dist(m[0],m[1],xEye,yEye) < 22;
  const nearObj = Math.abs(m[0]-xS)<16 && m[1]>(yS - hS/2 - 16) && m[1]<(yBaseS + 16);
  if (nearEye){ dragging="EYE"; grabDX=m[0]-xEye; grabDY=m[1]-yEye; }
  else if (nearObj){ dragging="S"; grabDX=m[0]-xS; grabDY=m[1]-yS; }
}
function mouseDragged(){
  if (!dragging) return;
  const m=fromCanvas(mouseX,mouseY);
  if (dragging==="EYE"){
    xEye=m[0]-grabDX; yEye=m[1]-grabDY;
    xEye=Math.min(xEye,-20); xEye=Math.max(xEye,-W/2+20);
    yEye=clamp(yEye,-H/2+20, H/2-20);
  } else if (dragging==="S"){
    xS=m[0]-grabDX; yS=m[1]-grabDY;
    xS=Math.min(xS,-40); xS=Math.max(xS,-W/2+40);
    const halfH=hS/2;
    yBaseS = yS + halfH; // chân đi theo khi kéo
    // giữ trong khung
    const topMin=-H/2+10, botMax=H/2-10;
    if ((yS - halfH) < topMin){ const d=topMin-(yS - halfH); yS+=d; yBaseS+=d; }
    if (yBaseS > botMax){ const d=yBaseS - botMax; yS-=d; yBaseS-=d; }
  }
}
function mouseReleased(){ dragging=null; }
</script>
</body>
</html>
