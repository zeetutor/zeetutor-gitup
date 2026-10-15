<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <title>Alphabet Line-Builder</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body, html { touch-action: none; user-select: none; -webkit-user-select: none; overflow: hidden; height: 100dvh; max-height: 100dvh; width: 100%; }
    .tray-piece, .mini-piece { transition: transform .15s ease-out; cursor: grab; }
    .tray-piece:active { cursor: grabbing; transform: scale(1.05); }
    .guide-slot.hovered, .mini-piece.hovered { filter: drop-shadow(0 0 6px #3b82f6); border-color: #3b82f6 !important; }
    #confetti-canvas { pointer-events:none; position:fixed; top:0; left:0; width:100%; height:100%; z-index:50; }
    .seg-bar { width:100%; height:100%; border-radius:6px; transition: background-color .2s; }
    .no-scrollbar::-webkit-scrollbar { display: none; }
    .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
  </style>
</head>
<body class="bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 h-[100dvh] max-h-[100dvh] text-slate-700 flex flex-col overflow-hidden font-sans">

<canvas id="confetti-canvas"></canvas>
<div id="drag-ghost" class="fixed pointer-events-none z-50 opacity-75 hidden flex items-center justify-center"></div>

<header class="bg-white/80 backdrop-blur-md shadow-sm border-b border-indigo-100/50 px-2.5 py-1 flex flex-wrap justify-between items-center gap-1 z-10 shrink-0">
  <div class="flex items-center gap-1.5">
    <div class="p-1 bg-gradient-to-br from-indigo-400 to-purple-500 text-white rounded-lg shadow">
      <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M12 6.042A8.967 8.967 0 006 3.75c-1.052 0-2.062.18-3 .512v14.25A8.987 8.987 0 016 18c2.305 0 4.408.867 6 2.292m0-14.25a8.966 8.966 0 016-2.292c1.052 0 2.062.18 3 .512v14.25A8.987 8.987 0 0018 18a8.967 8.967 0 00-6 2.292m0-14.25v14.25"/></svg>
    </div>
    <h1 class="font-black text-xs text-indigo-900 leading-none">Line-Builder</h1>
  </div>
  <div class="flex bg-slate-100 p-0.5 rounded-lg text-[10px]">
    <button id="tab-build" class="px-2 py-0.5 font-bold rounded-md text-indigo-600 bg-white shadow-sm" onclick="switchSection('build', this)">🔨 Build</button>
    <button id="tab-practice" class="px-2 py-0.5 font-bold rounded-md text-slate-500 hover:text-slate-800" onclick="switchSection('practice', this)">✏️ Practice</button>
    <button id="tab-canvas" class="px-2 py-0.5 font-bold rounded-md text-slate-500 hover:text-slate-800" onclick="switchSection('canvas', this)">🌈 Canvas</button>
  </div>
  <div class="hidden sm:flex items-center gap-1">
    <span id="room-badge" class="px-1.5 py-0.5 rounded-full text-[8px] font-mono font-bold bg-indigo-50 text-indigo-600 border border-indigo-100">Room: KidsZone</span>
  </div>
</header>

<main class="flex-1 overflow-hidden relative flex flex-col items-center justify-start pt-1.5 px-2 pb-2 gap-2">

  <section id="section-build" class="w-full max-w-lg mx-auto flex flex-col justify-start gap-1.5">
    <div class="flex items-center justify-between bg-white/80 p-1 rounded-xl gap-1 w-full max-w-xs mx-auto shadow-sm border border-indigo-100/50">
      <div class="flex items-center gap-0.5">
        <button id="btn-prev-letter" class="w-8 h-8 rounded-lg bg-white border border-slate-200 text-slate-600 flex items-center justify-center active:scale-95 text-xs font-bold shadow-sm" onclick="changeLetter(-1)">◀</button>
        <div class="bg-indigo-50/50 px-2 py-0.5 rounded-lg border border-indigo-100 flex items-center gap-1 min-w-[50px] justify-center h-8">
          <span id="active-letter-lbl" class="text-sm font-black text-indigo-600 font-mono">A</span>
        </div>
        <button id="btn-next-letter" class="w-8 h-8 rounded-lg bg-white border border-slate-200 text-slate-600 flex items-center justify-center active:scale-95 text-xs font-bold shadow-sm" onclick="changeLetter(1)">▶</button>
      </div>
      <div class="flex items-center gap-1">
        <button id="btn-toggle-trace" class="px-2.5 text-[10px] font-bold rounded-lg border bg-indigo-600 text-white border-indigo-500 h-8 shadow-sm" onclick="toggleTraceMode(this)">🔍 Trace: ON</button>
        <button id="btn-reset-letter" class="px-2.5 text-[10px] font-bold rounded-lg border border-pink-200 bg-pink-50 text-pink-600 h-8 shadow-sm" onclick="resetActiveLetter()">🔄 Reset</button>
      </div>
    </div>

    <div class="flex items-center justify-center relative">
      <div id="drop-box-arena" class="relative w-[min(70vw,32vh,270px)] h-[min(70vw,32vh,270px)] bg-white rounded-2xl shadow-md border-4 border-dashed border-indigo-200/60">
        <div class="absolute inset-0 rounded-xl opacity-10 pointer-events-none" style="background-image: radial-gradient(#4f46e5 1px, transparent 1px); background-size: 14px 14px;"></div>
        <div id="letter-complete-stamp" class="absolute inset-0 bg-emerald-500/95 backdrop-blur-sm rounded-xl flex flex-col justify-center items-center gap-1 opacity-0 pointer-events-none transition-all duration-300 z-10">
          <div class="w-10 h-10 bg-white text-emerald-500 rounded-full flex items-center justify-center text-xl shadow-md animate-bounce">✓</div>
          <h2 class="text-white font-black text-sm uppercase">Perfect!</h2>
        </div>
        <div id="blueprint-space" class="absolute inset-2 pointer-events-none"></div>
      </div>
    </div>

    <div class="w-full max-w-xs mx-auto bg-white/50 border border-indigo-100/60 rounded-xl p-1.5 shadow-sm">
      <div class="text-center text-[8.5px] font-black uppercase text-indigo-400 mb-1 tracking-wider">🎯 Match Pieces For This Letter</div>
      <div id="mini-grid-container" class="flex flex-wrap items-center justify-center gap-2 px-1 min-h-[48px]"></div>
    </div>
  </section>

  <section id="section-practice" class="w-full max-w-lg mx-auto flex flex-col justify-start gap-1.5 hidden">
    <div class="flex items-center justify-between bg-white/80 p-1 rounded-xl gap-1 w-full max-w-xs mx-auto shadow-sm border border-indigo-100/50">
      <div class="flex items-center gap-0.5">
        <button id="btn-prev-type" class="w-8 h-8 rounded-lg bg-white border border-slate-200 text-slate-600 flex items-center justify-center active:scale-95 text-xs font-bold" onclick="changePracticeType(-1)">◀</button>
        <div class="bg-indigo-50/50 px-2 py-0.5 rounded-lg border border-indigo-100 flex items-center gap-1 min-w-[80px] justify-center h-8">
          <span id="active-practice-lbl" class="text-[11px] font-black text-indigo-600 uppercase">Standing</span>
        </div>
        <button id="btn-next-type" class="w-8 h-8 rounded-lg bg-white border border-slate-200 text-slate-600 flex items-center justify-center active:scale-95 text-xs font-bold" onclick="changePracticeType(1)">▶</button>
      </div>
      <button id="btn-reset-practice" class="px-2.5 text-[10px] font-bold rounded-lg border border-pink-200 bg-pink-50 text-pink-600 h-8" onclick="resetPracticeGrid()">🔄 Reset</button>
    </div>
    <div class="grid grid-cols-4 gap-1.5 p-1 max-w-[280px] mx-auto w-full">
      <div id="practice-grid-container" class="contents"></div>
    </div>
    <div class="w-full max-w-xs mx-auto bg-white/50 border border-indigo-100/60 rounded-xl p-1.5 shadow-sm">
      <div class="text-center text-[8.5px] font-black uppercase text-indigo-400 mb-1 tracking-wider">🎯 Drag This Line</div>
      <div id="practice-piece-container" class="flex flex-wrap items-center justify-center gap-2 px-1 min-h-[48px]"></div>
    </div>
  </section>

  <section id="section-canvas" class="w-full max-w-lg mx-auto flex flex-col justify-start gap-1.5 hidden">
    <div class="flex items-center justify-between bg-white/80 p-1 rounded-xl gap-1 w-full max-w-xs mx-auto shadow-sm border border-indigo-100/50">
      <div class="flex items-center gap-1">
        <button id="btn-rainbow-toggle" class="px-2 py-1 text-[10px] font-bold rounded-lg border bg-gradient-to-r from-pink-500 via-yellow-400 to-blue-500 text-white h-8" onclick="toggleRainbow(this)">🌈 Rainbow</button>
        <button id="btn-canvas-trace-toggle" class="px-2 py-1 text-[10px] font-bold rounded-lg border bg-indigo-600 text-white border-indigo-500 h-8" onclick="toggleCanvasTrace(this)">🔍 Trace: ON</button>
      </div>
      <button id="btn-clear-canvas" class="px-2.5 text-[10px] font-bold rounded-lg border border-pink-200 bg-pink-50 text-pink-600 h-8" onclick="clearCanvas()">🧽 Clear</button>
    </div>
    <div class="flex items-center justify-center relative">
      <div class="relative w-[min(70vw,32vh,270px)] h-[min(70vw,32vh,270px)] bg-white rounded-2xl shadow-md border-4 border-dashed border-indigo-200/60 overflow-hidden">
        <div id="canvas-trace-guide" class="absolute inset-2 pointer-events-none opacity-[0.12]"></div>
        <canvas id="draw-canvas" class="absolute inset-0 w-full h-full" style="touch-action: none;"></canvas>
      </div>
    </div>
  </section>

</main>

<footer id="tray-footer" class="bg-white border-t border-indigo-100 pt-1 pb-[max(12px,env(safe-area-inset-bottom))] px-3 shrink-0 z-10 shadow-inner">
  <div class="max-w-4xl mx-auto">
    <div class="relative flex items-center">
      <button id="slide-left" class="absolute left-0 z-20 w-7 h-7 bg-white/95 border border-slate-200 shadow rounded-full flex items-center justify-center text-[10px] text-indigo-600 font-bold">◀</button>
      <div id="tray-list-viewport" class="w-full overflow-hidden mx-7">
        <div id="tray-list" class="flex gap-1.5 py-0.5 overflow-x-auto scroll-smooth no-scrollbar" style="-webkit-overflow-scrolling: touch;"></div>
      </div>
      <button id="slide-right" class="absolute right-0 z-20 w-7 h-7 bg-white/95 border border-slate-200 shadow rounded-full flex items-center justify-center text-[10px] text-indigo-600 font-bold">▶</button>
    </div>
  </div>
</footer>

<script>
let seed = 12345;
function seededRandom(){ let x = Math.sin(seed++) * 10000; return x - Math.floor(x); }
Math.random = seededRandom;

const STROKE_TYPES = ['standing','sleeping','slant-left','slant-right','curve-left','curve-right','zigzag','loop','wave'];
const BUILD_TYPES = ['standing','sleeping','slant-left','slant-right','curve-left','curve-right','curve-bottom'];

const LETTER_BLUEPRINTS = {
  A: [ {type:'slant-left', x1:50,y1:15,x2:25,y2:85}, {type:'slant-right', x1:50,y1:15,x2:75,y2:85}, {type:'sleeping', x1:33,y1:60,x2:67,y2:60} ],
  B: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'curve-right', bx:30,by:15,bw:35,bh:35}, {type:'curve-right', bx:30,by:50,bw:40,bh:35} ],
  C: [ {type:'curve-left', bx:30,by:15,bw:40,bh:70} ],
  D: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'curve-right', bx:30,by:15,bw:45,bh:70} ],
  E: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'sleeping', x1:30,y1:15,x2:65,y2:15}, {type:'sleeping', x1:30,y1:50,x2:55,y2:50}, {type:'sleeping', x1:30,y1:85,x2:65,y2:85} ],
  F: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'sleeping', x1:30,y1:15,x2:65,y2:15}, {type:'sleeping', x1:30,y1:50,x2:55,y2:50} ],
  G: [ {type:'curve-left', bx:30,by:15,bw:45,bh:70}, {type:'sleeping', x1:55,y1:55,x2:75,y2:55}, {type:'standing', x1:75,y1:55,x2:75,y2:85} ],
  H: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'standing', x1:70,y1:15,x2:70,y2:85}, {type:'sleeping', x1:30,y1:50,x2:70,y2:50} ],
  I: [ {type:'standing', x1:50,y1:15,x2:50,y2:85}, {type:'sleeping', x1:35,y1:15,x2:65,y2:15}, {type:'sleeping', x1:35,y1:85,x2:65,y2:85} ],
  J: [ {type:'sleeping', x1:35,y1:15,x2:75,y2:15}, {type:'standing', x1:65,y1:15,x2:65,y2:50}, {type:'curve-bottom', bx:35,by:50,bw:30,bh:35} ],
  K: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'slant-left', x1:65,y1:15,x2:30,y2:50}, {type:'slant-right', x1:30,y1:50,x2:65,y2:85} ],
  L: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'sleeping', x1:30,y1:85,x2:65,y2:85} ],
  M: [ {type:'standing', x1:20,y1:15,x2:20,y2:85}, {type:'standing', x1:80,y1:15,x2:80,y2:85}, {type:'slant-right', x1:20,y1:15,x2:50,y2:60}, {type:'slant-left', x1:80,y1:15,x2:50,y2:60} ],
  N: [ {type:'standing', x1:25,y1:15,x2:25,y2:85}, {type:'standing', x1:75,y1:15,x2:75,y2:85}, {type:'slant-right', x1:25,y1:15,x2:75,y2:85} ],
  O: [ {type:'curve-left', bx:25,by:15,bw:25,bh:70}, {type:'curve-right', bx:50,by:15,bw:25,bh:70} ],
  P: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'curve-right', bx:30,by:15,bw:40,bh:35} ],
  Q: [ {type:'curve-left', bx:25,by:15,bw:25,bh:70}, {type:'curve-right', bx:50,by:15,bw:25,bh:70}, {type:'slant-right', x1:55,y1:65,x2:80,y2:90} ],
  R: [ {type:'standing', x1:30,y1:15,x2:30,y2:85}, {type:'curve-right', bx:30,by:15,bw:40,bh:35}, {type:'slant-right', x1:30,y1:50,x2:65,y2:85} ],
  S: [ {type:'curve-left', bx:30,by:15,bw:40,bh:35}, {type:'curve-right', bx:30,by:50,bw:40,bh:35} ],
  T: [ {type:'sleeping', x1:20,y1:15,x2:80,y2:15}, {type:'standing', x1:50,y1:15,x2:50,y2:85} ],
  U: [ {type:'standing', x1:25,y1:15,x2:25,y2:60}, {type:'standing', x1:75,y1:15,x2:75,y2:60}, {type:'curve-bottom', bx:25,by:60,bw:50,bh:25} ],
  V: [ {type:'slant-right', x1:25,y1:15,x2:50,y2:85}, {type:'slant-left', x1:75,y1:15,x2:50,y2:85} ],
  W: [ {type:'slant-right', x1:15,y1:15,x2:35,y2:85}, {type:'slant-left', x1:50,y1:45,x2:35,y2:85}, {type:'slant-right', x1:50,y1:45,x2:65,y2:85}, {type:'slant-left', x1:85,y1:15,x2:65,y2:85} ],
  X: [ {type:'slant-right', x1:22,y1:15,x2:78,y2:85}, {type:'slant-left', x1:78,y1:15,x2:22,y2:85} ],
  Y: [ {type:'slant-right', x1:25,y1:15,x2:50,y2:50}, {type:'slant-left', x1:75,y1:15,x2:50,y2:50}, {type:'standing', x1:50,y1:50,x2:50,y2:85} ],
  Z: [ {type:'sleeping', x1:25,y1:15,x2:75,y2:15}, {type:'slant-left', x1:75,y1:15,x2:25,y2:85}, {type:'sleeping', x1:25,y1:85,x2:75,y2:85} ]
};
Object.keys(LETTER_BLUEPRINTS).forEach(letter => { LETTER_BLUEPRINTS[letter].forEach((seg, i) => seg.id = `${letter}-${i}`); });

const LETTERS = Object.keys(LETTER_BLUEPRINTS);
let activeLetterIdx = 0;
let traceGuideOn = true;
let activeSection = 'build';
let placedBuildStrokes = {};
let practiceTypeIdx = 0;
let placedPracticeCells = {};

function getStrokeSVG(type, color, thickness='6px'){
  switch(type){
    case 'curve-left': return `<div style="box-sizing:border-box; border: ${thickness} solid ${color}; border-right:0; border-radius:1000px 0 0 1000px; width:100%; height:100%;"></div>`;
    case 'curve-right': return `<div style="box-sizing:border-box; border: ${thickness} solid ${color}; border-left:0; border-radius:0 1000px 1000px 0; width:100%; height:100%;"></div>`;
    case 'curve-bottom': return `<div style="box-sizing:border-box; border: ${thickness} solid ${color}; border-top:0; border-radius:0 0 1000px 1000px; width:100%; height:100%;"></div>`;
    case 'standing': return `<svg class="w-full h-full" viewBox="0 0 20 100"><rect x="10" y="5" width="8" height="90" transform="translate(-4,0)" rx="4" fill="${color}"/></svg>`;
    case 'sleeping': return `<svg class="w-full h-full" viewBox="0 0 100 20"><rect x="5" y="10" width="90" height="8" transform="translate(0,-4)" rx="4" fill="${color}"/></svg>`;
    case 'slant-left': return `<svg class="w-full h-full" viewBox="0 0 100 100"><line x1="85" y1="15" x2="15" y2="85" stroke="${color}" stroke-width="${parseInt(thickness)}" stroke-linecap="round"/></svg>`;
    case 'slant-right': return `<svg class="w-full h-full" viewBox="0 0 100 100"><line x1="15" y1="15" x2="85" y2="85" stroke="${color}" stroke-width="${parseInt(thickness)}" stroke-linecap="round"/></svg>`;
    case 'zigzag': return `<svg class="w-full h-full" viewBox="0 0 100 100"><path d="M 15,50 L 38,20 L 62,80 L 85,50" fill="none" stroke="${color}" stroke-width="${parseInt(thickness)}" stroke-linecap="round" stroke-linejoin="round"/></svg>`;
    case 'loop': return `<svg class="w-full h-full" viewBox="0 0 100 100"><path d="M 15,65 C 45,65 35,25 50,25 C 65,25 55,65 85,65" fill="none" stroke="${color}" stroke-width="${parseInt(thickness)}" stroke-linecap="round"/></svg>`;
    case 'wave': return `<svg class="w-full h-full" viewBox="0 0 100 100"><path d="M 15,50 Q 32.5,20 50,50 T 85,50" fill="none" stroke="${color}" stroke-width="${parseInt(thickness)}" stroke-linecap="round"/></svg>`;
    default: return '';
  }
}
function friendlyLabel(t){ return t.replace('-', ' '); }

let audioCtx = null;
function initAudio(){ if(!audioCtx) audioCtx = new (window.AudioContext||window.webkitAudioContext)(); }
function playTone(freq,type,dur){
  initAudio(); if(!audioCtx) return;
  try{
    const osc=audioCtx.createOscillator(), gain=audioCtx.createGain();
    osc.type=type||'sine'; osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
    gain.gain.setValueAtTime(0.12, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime+dur);
    osc.connect(gain); gain.connect(audioCtx.destination);
    osc.start(); osc.stop(audioCtx.currentTime+dur);
  }catch(e){}
}
function playSfxMatch(){ playTone(523.25,'triangle',0.1); setTimeout(()=>playTone(659.25,'triangle',0.15),80); }
function playSfxWrong(){ playTone(160,'sawtooth',0.23); }
function playSfxPop(){ playTone(783.99,'sine',0.08); }
function playSfxWin(){ [523.25,587.33,659.25,698.46,783.99,880,987.77,1046.5].forEach((f,i)=>setTimeout(()=>playTone(f,'triangle',0.12), i*55)); }

function lineGeom(seg){
  const dx = seg.x2-seg.x1, dy = seg.y2-seg.y1;
  return { len: Math.hypot(dx,dy), angle: Math.atan2(dy,dx)*180/Math.PI, midX:(seg.x1+seg.x2)/2, midY:(seg.y1+seg.y2)/2 };
}

function buildTray(){
  // Bottom footer tray is retired entirely - Build uses the mini-grid under its
  // arena, Practice uses its own under-grid palette. Keeping this function name
  // so existing calls (switchSection, changeLetter, etc.) don't need touching.
  document.getElementById('tray-footer').classList.add('hidden');
  if(activeSection==='build'){ populateMiniGrid(); }
  else if(activeSection==='practice'){ populatePracticePalette(); }
}

function populatePracticePalette(){
  const container = document.getElementById('practice-piece-container');
  if(!container) return;
  container.innerHTML = '';
  // Always show all 9 line types, same consistent pattern as the Build tab's mini-grid.
  // The grid above only accepts the currently-selected type; wrong pieces just bounce.
  STROKE_TYPES.forEach(type => {
    const box = document.createElement('div');
    box.className = "mini-piece w-11 h-11 rounded-xl flex items-center justify-center shadow-sm transition-all relative bg-white border-2 border-dashed border-slate-200 hover:border-indigo-300 text-slate-300";
    box.dataset.type = type;
    box.style.touchAction = "none";
    let wrapClasses = "w-6 h-6";
    if (type.includes('curve-left') || type.includes('curve-right')) wrapClasses = "w-3.5 h-6";
    if (type === 'curve-bottom') wrapClasses = "w-6 h-3.5";
    box.innerHTML = `<div class="${wrapClasses} flex items-center justify-center pointer-events-none">${getStrokeSVG(type, '#cbd5e1', "3.5px")}</div>`;
    box.addEventListener('pointerdown', e => startCustomDrag(e, type));
    container.appendChild(box);
  });
}

function populateMiniGrid() {
  const miniGrid = document.getElementById('mini-grid-container');
  if (!miniGrid) return;
  miniGrid.innerHTML = '';
  // Always show all 7 line types, same set as the tray, regardless of which letter is active.
  BUILD_TYPES.forEach(type => {
    const box = document.createElement('div');
    box.className = "mini-piece w-11 h-11 rounded-xl flex items-center justify-center shadow-sm transition-all relative bg-white border-2 border-dashed border-slate-200 hover:border-indigo-300 text-slate-300";
    box.dataset.type = type;
    box.style.touchAction = "none";
    let wrapClasses = "w-6 h-6";
    if (type.includes('curve-left') || type.includes('curve-right')) wrapClasses = "w-3.5 h-6";
    if (type === 'curve-bottom') wrapClasses = "w-6 h-3.5";
    box.innerHTML = `<div class="${wrapClasses} flex items-center justify-center pointer-events-none">${getStrokeSVG(type, '#cbd5e1', "3.5px")}</div>`;
    box.addEventListener('pointerdown', e => startCustomDrag(e, type));
    miniGrid.appendChild(box);
  });
}

document.getElementById('slide-left').onclick = () => { document.getElementById('tray-list').scrollBy({ left: -90, behavior: 'smooth' }); };
document.getElementById('slide-right').onclick = () => { document.getElementById('tray-list').scrollBy({ left: 90, behavior: 'smooth' }); };

function renderActiveLetter(){
  const letter = LETTERS[activeLetterIdx];
  const labelEl = document.getElementById('active-letter-lbl');
  if(labelEl) labelEl.innerText = letter;
  const space = document.getElementById('blueprint-space');
  if(!space) return;
  space.innerHTML = '';
  document.getElementById('letter-complete-stamp').style.opacity = '0';
  const blueprint = LETTER_BLUEPRINTS[letter] || [];
  blueprint.forEach(seg=>{
    const isPlaced = !!placedBuildStrokes[seg.id];
    const color = isPlaced ? '#6366f1' : (traceGuideOn ? '#cbd5e1' : 'transparent');
    const slot = document.createElement('div');
    slot.id = `slot-${seg.id}`;
    slot.className = "guide-slot absolute pointer-events-none";
    slot.dataset.id = seg.id;
    slot.dataset.type = seg.type;
    if(seg.type.includes('curve')){
      slot.style.left = seg.bx+'%'; slot.style.top = seg.by+'%';
      slot.style.width = seg.bw+'%'; slot.style.height = seg.bh+'%';
      slot.innerHTML = getStrokeSVG(seg.type, color, '10px');
    } else {
      const g = lineGeom(seg);
      slot.style.left = g.midX+'%'; slot.style.top = g.midY+'%';
      slot.style.width = g.len+'%'; slot.style.height = '10px';
      slot.style.transform = `translate(-50%,-50%) rotate(${g.angle}deg)`;
      const bar = document.createElement('div');
      bar.className = 'seg-bar';
      bar.style.background = color;
      slot.appendChild(bar);
    }
    space.appendChild(slot);
  });
  populateMiniGrid();
  checkBuildCompletion(false);
}

function renderPracticeGrid(){
  const type = STROKE_TYPES[practiceTypeIdx];
  document.getElementById('active-practice-lbl').innerText = friendlyLabel(type);
  const container = document.getElementById('practice-grid-container');
  if(!container) return;
  container.innerHTML = '';
  for(let i=0; i<8; i++){
    const isPlaced = !!placedPracticeCells[i];
    const cell = document.createElement('div');
    cell.id = `practice-cell-${i}`;
    cell.className = "practice-slot relative bg-white rounded-xl border-2 border-slate-200 p-1 flex items-center justify-center shadow-sm aspect-square min-h-[44px] min-w-[44px] " + (isPlaced ? "bg-emerald-50/40 border-emerald-300" : "");
    cell.dataset.index = i;
    cell.dataset.type = type;
    const area = document.createElement('div');
    area.className = "pointer-events-none flex items-center justify-center w-full h-full p-1";
    let wrapClasses = "w-full h-full";
    if (type.includes('curve-left') || type.includes('curve-right')) wrapClasses = "w-6 h-10";
    if (type === 'curve-bottom') wrapClasses = "w-10 h-6";
    area.innerHTML = `<div class="${wrapClasses}">${getStrokeSVG(type, isPlaced ? '#10b981' : '#d8dee9')}</div>`;
    cell.appendChild(area);
    if(isPlaced){
      const chk = document.createElement('div');
      chk.className = "absolute top-1 right-1.5 text-emerald-500 font-bold text-xs pointer-events-none";
      chk.innerText = "✓";
      cell.appendChild(chk);
    }
    container.appendChild(cell);
  }
}

function switchSection(section, btn){
  if(window.isSimulating) return;
  activeSection = section;
  ['tab-build','tab-practice','tab-canvas'].forEach(id=>{
    document.getElementById(id).className = "px-2.5 py-0.5 font-bold rounded-md text-slate-500 hover:text-slate-800";
  });
  btn.className = "px-2.5 py-0.5 font-bold rounded-md text-indigo-600 bg-white shadow-sm";
  document.getElementById('section-build').classList.toggle('hidden', section!=='build');
  document.getElementById('section-practice').classList.toggle('hidden', section!=='practice');
  document.getElementById('section-canvas').classList.toggle('hidden', section!=='canvas');
  if(section==='practice') renderPracticeGrid();
  if(section==='canvas') initCanvasSection();
  buildTray();
  sendSyncClick(btn);
}

function toggleTraceMode(btn){
  if(window.isSimulating) return;
  traceGuideOn = !traceGuideOn;
  btn.innerText = traceGuideOn ? "🔍 Trace: ON" : "🔍 Trace: OFF";
  btn.className = "px-2.5 text-[11px] font-bold rounded-lg border shadow-sm h-8 " + (traceGuideOn ? "bg-indigo-600 text-white border-indigo-500" : "bg-white text-slate-600 border-slate-200");
  renderActiveLetter();
  sendSyncClick(btn);
}

function changeLetter(dir){
  if(window.isSimulating) return;
  activeLetterIdx = (activeLetterIdx+dir+LETTERS.length)%LETTERS.length;
  placedBuildStrokes = {};
  renderActiveLetter();
  sendSyncClick(document.getElementById(dir>0?'btn-next-letter':'btn-prev-letter'));
}

function changePracticeType(dir){
  if(window.isSimulating) return;
  practiceTypeIdx = (practiceTypeIdx+dir+STROKE_TYPES.length)%STROKE_TYPES.length;
  placedPracticeCells = {};
  renderPracticeGrid();
  buildTray();
  sendSyncClick(document.getElementById(dir>0?'btn-next-type':'btn-prev-type'));
}

function resetActiveLetter(){ if(window.isSimulating) return; placedBuildStrokes={}; renderActiveLetter(); sendSyncClick(document.getElementById('btn-reset-letter')); }
function resetPracticeGrid(){ if(window.isSimulating) return; placedPracticeCells={}; renderPracticeGrid(); sendSyncClick(document.getElementById('btn-reset-practice')); }

let activeDragType = null, dragGhost = null, lastCursorSend = 0;

function startCustomDrag(e, type){
  initAudio(); e.preventDefault();
  try{ e.target.setPointerCapture(e.pointerId); }catch(err){}
  activeDragType = type;
  dragGhost = document.getElementById('drag-ghost');
  let wrapClasses = "w-full h-full";
  if (type.includes('curve-left') || type.includes('curve-right')) wrapClasses = "w-6 h-12";
  if (type === 'curve-bottom') wrapClasses = "w-12 h-6";
  dragGhost.innerHTML = `<div class="${wrapClasses}">${getStrokeSVG(type, '#6366f1')}</div>`;
  dragGhost.className = "fixed pointer-events-none z-50 opacity-70 w-12 h-12 flex items-center justify-center";
  dragGhost.style.left = e.clientX+'px'; dragGhost.style.top = e.clientY+'px';
  dragGhost.style.transform = `translate(-50%, -50%)`;
  dragGhost.classList.remove('hidden');
  window.addEventListener('pointermove', onCustomDragMove);
  window.addEventListener('pointerup', onCustomDragUp);
  window.addEventListener('pointercancel', onCustomDragUp);
}

function locateDropTarget(e, selector, isPlacedFn){
  const els = document.querySelectorAll(selector);
  let best=null, bestArea=Infinity;
  els.forEach(el=>{
    if(isPlacedFn(el)) return;
    const r = el.getBoundingClientRect();
    const pad = 28;
    if(e.clientX>=r.left-pad && e.clientX<=r.right+pad && e.clientY>=r.top-pad && e.clientY<=r.bottom+pad){
      const area = r.width*r.height;
      if(area<bestArea){ bestArea=area; best=el; }
    }
  });
  return best;
}

function onCustomDragMove(e){
  if(!dragGhost) return;
  dragGhost.style.left = e.clientX+'px'; dragGhost.style.top = e.clientY+'px';
  document.querySelectorAll('.guide-slot,.practice-slot,.mini-piece').forEach(s=>s.classList.remove('hovered'));
  let target=null;
  if(activeSection==='build') {
    target = locateDropTarget(e, '.guide-slot', el => placedBuildStrokes[el.dataset.id]);
  } else if(activeSection==='practice') {
    target = locateDropTarget(e, '.practice-slot', el => placedPracticeCells[parseInt(el.dataset.index)]);
  }
  if(target && target.dataset.type===activeDragType) target.classList.add('hovered');
}

function onCustomDragUp(e){
  window.removeEventListener('pointermove', onCustomDragMove);
  window.removeEventListener('pointerup', onCustomDragUp);
  window.removeEventListener('pointercancel', onCustomDragUp);
  if(!dragGhost) return;
  dragGhost.classList.add('hidden'); dragGhost=null;
  let matched=false;
  if(activeSection==='build'){
    let target = locateDropTarget(e, '.guide-slot', el => placedBuildStrokes[el.dataset.id]);
    if (target && target.dataset.type===activeDragType) {
      matched = true;
      placeStrokeLocal(target.id, target.dataset.id, activeDragType);
    }
  } else if(activeSection==='practice'){
    const target = locateDropTarget(e,'.practice-slot', el=>placedPracticeCells[parseInt(el.dataset.index)]);
    if(target && target.dataset.type===activeDragType){ matched=true; placePracticeLocal(target.id, parseInt(target.dataset.index), activeDragType); }
  }
  if(!matched) playSfxWrong();
  activeDragType = null;
}

function placeStrokeLocal(slotId, strokeId, type){
  placedBuildStrokes[strokeId] = true;
  playSfxMatch();
  renderActiveLetter();
  if(!window.isSimulating){
    window.parent.postMessage({type:'IFRAME_ACTION_TRIGGERED', payload:{ type:'LINE_PLACED', selector:`#${slotId}`, value:type, actionId: crypto.randomUUID() }}, '*');
  }
}
function placePracticeLocal(cellId, index, type){
  placedPracticeCells[index] = true;
  playSfxPop();
  renderPracticeGrid();
}
function checkBuildCompletion(withCeleb=true){
  const letter = LETTERS[activeLetterIdx];
  const bp = LETTER_BLUEPRINTS[letter]||[];
  const done = bp.length>0 && bp.every(s=>placedBuildStrokes[s.id]);
  if(done){
    document.getElementById('letter-complete-stamp').style.opacity = '1';
    document.getElementById('letter-complete-stamp').classList.remove('pointer-events-none');
    if(withCeleb){
      playSfxWin(); startConfetti();
      setTimeout(()=>{ if(activeSection==='build') changeLetter(1); }, 3200);
    }
  }
}
function sendSyncClick(el){
  if(window.isSimulating||!el) return;
  window.parent.postMessage({type:'IFRAME_ACTION_TRIGGERED', payload:{ type:'UNIVERSAL_CLICK', selector:el.id ? `#${el.id}` : el.tagName.toLowerCase(), actionId: crypto.randomUUID() }}, '*');
}

let canvas, ctx, rainbowOn=true, canvasTraceOn=true, hue=0, drawing=false, lastPct=null, lastStrokeSend=0;

function initCanvasSection(){
  canvas = document.getElementById('draw-canvas');
  if(!canvas) return;
  ctx = canvas.getContext('2d');
  canvas.style.touchAction = 'none';
  const rect = canvas.getBoundingClientRect();
  canvas.width = rect.width; canvas.height = rect.height;
  renderCanvasTraceGuide();
  canvas.onpointerdown = e => handleStrokeStart(e, true);
  canvas.onpointermove = e => handleStrokeMove(e, true);
}

function renderCanvasTraceGuide(){
  const guide = document.getElementById('canvas-trace-guide');
  if(!guide) return;
  guide.innerHTML = '';
  if(!canvasTraceOn) return;
  const letter = LETTERS[activeLetterIdx];
  (LETTER_BLUEPRINTS[letter]||[]).forEach(seg=>{
    const el = document.createElement('div');
    el.className = "absolute";
    if(seg.type.includes('curve')){
      el.style.left=seg.bx+'%'; el.style.top=seg.by+'%'; el.style.width=seg.bw+'%'; el.style.height=seg.bh+'%';
      el.innerHTML = getStrokeSVG(seg.type, '#334155', '10px');
    } else {
      const g = lineGeom(seg);
      el.style.left=g.midX+'%'; el.style.top=g.midY+'%'; el.style.width=g.len+'%'; el.style.height='10px';
      el.style.transform = `translate(-50%,-50%) rotate(${g.angle}deg)`;
      el.style.background = '#334155'; el.style.borderRadius = '6px';
    }
    guide.appendChild(el);
  });
}

function toggleRainbow(btn){ rainbowOn = !rainbowOn; btn.innerText = rainbowOn ? "🌈 Rainbow" : "🖊️ Fixed Color"; }
function toggleCanvasTrace(btn) { canvasTraceOn = !canvasTraceOn; btn.innerText = canvasTraceOn ? "🔍 Trace: ON" : "🔍 Trace: OFF"; renderCanvasTraceGuide(); }
function clearCanvas(){ if(ctx) ctx.clearRect(0,0,canvas.width,canvas.height); }

function handleStrokeStart(e, isLocal){
  initAudio(); e.preventDefault?.();
  try{ canvas.setPointerCapture(e.pointerId); }catch(err){}
  const rect = canvas.getBoundingClientRect();
  const pctX = ((e.clientX-rect.left)/rect.width)*100, pctY = ((e.clientY-rect.top)/rect.height)*100;
  drawing = true; lastPct = {pctX,pctY};
}
function handleStrokeMove(e, isLocal){
  if(!drawing || !lastPct) return;
  const rect = canvas.getBoundingClientRect();
  const pctX = ((e.clientX-rect.left)/rect.width)*100, pctY = ((e.clientY-rect.top)/rect.height)*100;
  const p1 = pctToPx(lastPct.pctX,lastPct.pctY), p2 = pctToPx(pctX,pctY);
  ctx.strokeStyle = rainbowOn ? `hsl(${(hue+=3)%360},100%,50%)` : '#4f46e5';
  ctx.lineWidth = 5; ctx.lineCap = 'round'; ctx.lineJoin='round';
  ctx.beginPath(); ctx.moveTo(p1.x,p1.y); ctx.lineTo(p2.x,p2.y); ctx.stroke();
  lastPct = {pctX,pctY};
}
function pctToPx(pctX,pctY){ return { x:(pctX/100)*canvas.width, y:(pctY/100)*canvas.height }; }

window.addEventListener('message', e=>{
  const d = e.data;
  if(d?.type !== 'EXECUTE_UNIVERSAL_ACTION') return;
  const action = d.action;
  const el = document.querySelector(action.selector);
  if(!el) return;
  window.isSimulating = true;
  el.click();
  window.isSimulating = false;
});

function startConfetti(){
  const canvasC = document.getElementById('confetti-canvas');
  canvasC.width = window.innerWidth; canvasC.height = window.innerHeight;
  const c = canvasC.getContext('2d');
  const colors = ['#f43f5e','#ec4899','#a855f7','#6366f1','#3b82f6'];
  const particles = Array.from({length:70}, ()=>({
    x: Math.random()*canvasC.width, y: Math.random()*canvasC.height-canvasC.height,
    r: Math.random()*5+3, d: Math.random()*canvasC.height, color: colors[Math.floor(Math.random()*colors.length)],
    tilt: Math.random()*10-5, tai: Math.random()*0.05+0.01, ta:0
  }));
  const start = Date.now();
  function draw(){
    c.clearRect(0,0,canvasC.width,canvasC.height);
    particles.forEach((p,i)=>{
      p.ta += p.tai; p.y += (Math.cos(p.d)+2+p.r/2)/2; p.x += Math.sin(p.ta);
      c.beginPath(); c.lineWidth=p.r; c.strokeStyle=p.color;
      c.moveTo(p.x+p.tilt, p.y); c.lineTo(p.x, p.y+p.tilt); c.stroke();
    });
    if(Date.now()-start<2500) requestAnimationFrame(draw); else c.clearRect(0,0,canvasC.width,canvasC.height);
  }
  draw();
}

window.onload = function(){ buildTray(); renderActiveLetter(); };
</script>
</body>
</html>
