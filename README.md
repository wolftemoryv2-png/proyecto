<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Recorrido Virtual – UMB Ixtapaluca</title>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/pannellum/build/pannellum.css">
<script src="https://cdn.jsdelivr.net/npm/pannellum/build/pannellum.js"></script>

<style>
body{
    margin:0;
    font-family:'Segoe UI',Arial,sans-serif;
    background: linear-gradient(to bottom,#f1f8f4,#ffffff);
}

/* HEADER */
header{
    background:#AAA;
    height:60px;
    border-bottom:1px solid #ddd;
    display:flex;
    align-items:center;
    justify-content:center;
    position:relative;
}
header img{height:40px;}
header h1{font-size:18px;font-weight:500;margin:0;}

/* VISOR */
#panorama{
    width:100%;
    height:calc(100vh - 160px);
}

/* NOMBRE ESCENA */
#sceneName{
    position:fixed;
    top:75px;
    left:50%;
    transform:translateX(-50%);
    background:rgba(255,255,255,0.9);
    padding:8px 18px;
    border-radius:20px;
    font-size:15px;
    font-weight:500;
    box-shadow:0 3px 8px rgba(0,0,0,0.25);
    z-index:40;
    opacity:0;
    transition:opacity .5s;
}
#sceneName.show{opacity:1;}

/* CONTROLES DE MOVIMIENTO */
.controls{
    position:fixed;
    bottom:95px;
    left:50%;
    transform:translateX(-50%);
    display:flex;
    flex-direction:column;
    align-items:center;
    gap:6px;
    z-index:30;
    opacity:0;
    pointer-events:none;
    transition:opacity .4s;
}
.controls.visible{opacity:1;pointer-events:auto;}
.controls-row{display:flex;gap:40px;}
.controls button{
    width:48px;
    height:48px;
    border-radius:50%;
    border:none;
    background:rgba(0,0,0,.65);
    color:#fff;
    font-size:20px;
    cursor:pointer;
}
.controls button:active{transform:scale(.95);}

/* AUDIO CONTROLES */
.audio-controls{
    position:fixed;
    bottom:25px;
    left:25px;
    display:flex;
    gap:10px;
    z-index:30;
    opacity:0;
    pointer-events:none;
    transition:opacity .4s;
}
.audio-controls.visible{opacity:1;pointer-events:auto;}
.audio-controls button{
    width:42px;
    height:42px;
    border-radius:50%;
    border:none;
    background:rgba(0,0,0,.6);
    color:#fff;
    font-size:18px;
    cursor:pointer;
}
.audio-controls button:active{transform:scale(.95);}

/* MINIATURAS */
.thumbs{
    position:fixed;
    bottom:20px;
    left:50%;
    transform:translateX(-50%);
    display:flex;
    gap:10px;
    background:rgba(255,255,255,.95);
    padding:10px 14px;
    border-radius:30px;
    box-shadow:0 4px 12px rgba(0,0,0,.25);
    z-index:25;
}
.thumb img{
    width:90px;
    height:55px;
    object-fit:cover;
    border-radius:6px;
    border:2px solid transparent;
    cursor:pointer;
}
.thumb.active img{border:2px solid #2e7d32;}
</style>
</head>

<body>

<header>
    <img src="imagenes/logo.png" style="position:absolute;left:15px;">
    <h1>Universidad Mexiquense del Bicentenario – UES Ixtapaluca</h1>
    <img src="imagenes/edo.png" style="position:absolute;right:15px;">
</header>

<div id="panorama"></div>
<div id="sceneName"></div>

<!-- CONTROLES -->
<div class="controls" id="controls">
    <button id="up">▲</button>
    <div class="controls-row">
        <button id="left">◀</button>
        <button id="right">▶</button>
    </div>
    <button id="down">▼</button>
</div>

<!-- AUDIO -->
<div class="audio-controls" id="audioControls">
    <button id="volDown">🔉</button>
    <button id="mute">🔇</button>
    <button id="volUp">🔊</button>
</div>

<!-- MINIATURAS -->
<div class="thumbs" id="thumbContainer"></div>

<audio id="audioAmbiente" loop>
    <source src="audio/musica.mp3" type="audio/mpeg">
</audio>

<script>
const totalFotos=12;
const EXT=".jpg";

const nombresEscenas=[
"Entrada Principal","Área General del Campus","Patio del Edificio A",
"Pasillo de Conexión","Cafetería y Área de Cultivo","Edificio B",
"Salón de Ingeniería Agrícola","Pasillo Interior",
"Campo de Cultivo Experimental","Pasillo Exterior",
"Área de Jardineras","Pasillo hacia el Estacionamiento"
];

const VELOCIDAD=2.2, PASO=25, LIMITE_PITCH=85;

const controls=document.getElementById("controls");
const audioControls=document.getElementById("audioControls");
let hideTimeout;

function mostrarControles(){
    controls.classList.add("visible");
    audioControls.classList.add("visible");
    clearTimeout(hideTimeout);
    hideTimeout=setTimeout(()=>{
        controls.classList.remove("visible");
        audioControls.classList.remove("visible");
    },4000);
}

const viewer=pannellum.viewer('panorama',{
    default:{firstScene:"s1",sceneFadeDuration:800},
    scenes:{}
});

/* 🔑 FIX: nombre SIEMPRE visible al cambiar de escena */
viewer.on("scenechange",function(sceneId){
    const index=parseInt(sceneId.replace("s",""))-1;
    const t=document.getElementById("sceneName");

    t.textContent=nombresEscenas[index];
    t.classList.add("show");

    clearTimeout(t.hideTimer);
    t.hideTimer=setTimeout(()=>t.classList.remove("show"),4000);

    document.querySelectorAll(".thumb").forEach(el=>el.classList.remove("active"));
    const thumb=document.getElementById("thumb"+(index+1));
    if(thumb) thumb.classList.add("active");
});

const anterior=n=>n===1?totalFotos:n-1;
const siguiente=n=>n===totalFotos?1:n+1;

for(let i=1;i<=totalFotos;i++){
    viewer.addScene("s"+i,{
        type:"equirectangular",
        panorama:"imagenes/"+i+EXT,
        hfov:110,
        hotSpots:[
            {pitch:0,yaw:-40,type:"scene",sceneId:"s"+anterior(i)},
            {pitch:0,yaw:40,type:"scene",sceneId:"s"+siguiente(i)}
        ]
    });
}

const cont=document.getElementById("thumbContainer");
for(let i=1;i<=totalFotos;i++){
    const d=document.createElement("div");
    d.className="thumb"; d.id="thumb"+i;
    d.innerHTML=`<img src="imagenes/${i}.jpg">`;
    d.onclick=()=>viewer.loadScene("s"+i);
    cont.appendChild(d);
}

/* MOVIMIENTO */
let h=null,v=null;
const startH=d=>{if(!h)h=setInterval(()=>viewer.setYaw(viewer.getYaw()+d*VELOCIDAD),16);}
const stopH=()=>{clearInterval(h);h=null;}
const startV=d=>{if(!v)v=setInterval(()=>{
    let p=viewer.getPitch()+d*VELOCIDAD;
    viewer.setPitch(Math.max(-LIMITE_PITCH,Math.min(LIMITE_PITCH,p)));
},16);}
const stopV=()=>{clearInterval(v);v=null;}

left.onclick=()=>viewer.setYaw(viewer.getYaw()-PASO);
right.onclick=()=>viewer.setYaw(viewer.getYaw()+PASO);
up.onclick=()=>viewer.setPitch(Math.min(viewer.getPitch()+PASO,LIMITE_PITCH));
down.onclick=()=>viewer.setPitch(Math.max(viewer.getPitch()-PASO,-LIMITE_PITCH));

left.onmousedown=()=>startH(-1);
right.onmousedown=()=>startH(1);
up.onmousedown=()=>startV(1);
down.onmousedown=()=>startV(-1);

[left,right,up,down].forEach(b=>{
    b.onmouseup=b.onmouseleave=()=>{stopH();stopV();}
});

/* AUDIO */
const audio=document.getElementById("audioAmbiente");
let ultimoVol=0.25;
document.addEventListener("click",()=>{
    if(audio.paused){
        audio.volume=ultimoVol;
        audio.play();
    }
},{once:true});

volUp.onclick=()=>{audio.volume=Math.min(1,audio.volume+.1);audio.muted=false;}
volDown.onclick=()=>{audio.volume=Math.max(0,audio.volume-.1);}
mute.onclick=()=>{
    if(audio.muted){
        audio.muted=false;
        audio.volume=ultimoVol;
        mute.textContent="🔇";
    }else{
        ultimoVol=audio.volume;
        audio.muted=true;
        mute.textContent="🔈";
    }
};

["mousemove","touchstart","click"].forEach(e=>{
    document.addEventListener(e,mostrarControles);
});

window.onload=()=>{viewer.loadScene("s1");mostrarControles();}
</script>

</body>
</html>
