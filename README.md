<!DOCTYPE html>
<html>
<head>
<style>
* { margin:0; padding:0; box-sizing:border-box; }
body { background:#1a1a1a; display:flex; align-items:center; justify-content:center; height:100vh; overflow:hidden; user-select:none; }
canvas { display:block; max-width:100%; max-height:100%; }
</style>
</head>
<body>
<canvas id="c"></canvas>
<script>
const c = document.getElementById('c');
const ctx = c.getContext('2d');
const W = 800, H = 450;
c.width = W; c.height = H;

// NC State palette
const R='#CC0000', W2='#fff', BK='#111', GD='#FFD700', OR='#FF6600', BL='#44aaff';

// Track geometry
const TY=118, TH=240, LH=TH/3;
const LY=[0,1,2].map(i=>TY+i*LH+LH/2);

let g;
function reset() {
  g = {
    s:'start', score:0, dist:0, spd:4, fr:0,
    stam:100, hoffs:0,
    lane:1, ry:LY[1], rf:0, cd:0,
    objs:[], nextObj:350,
    hz:null, nextHz:900,
    sx:0, mx:0,
    hitFlash:0,
  };
}

function press(k) {
  if (g.s!=='play') { reset(); g.s='play'; return; }
  if (g.cd>0) return;
  if ((k==='ArrowUp'||k==='w'||k==='W') && g.lane>0)  { g.lane--; g.cd=12; }
  if ((k==='ArrowDown'||k==='s'||k==='S') && g.lane<2) { g.lane++; g.cd=12; }
}

document.addEventListener('keydown', e => {
  if(['ArrowUp','ArrowDown'].includes(e.key)) e.preventDefault();
  press(e.key);
});
let ty0=0;
c.addEventListener('touchstart', e=>{ ty0=e.touches[0].clientY; e.preventDefault(); },{passive:false});
c.addEventListener('touchend', e=>{
  const d=e.changedTouches[0].clientY-ty0;
  press(Math.abs(d)<15?' ':d<0?'ArrowUp':'ArrowDown');
  e.preventDefault();
},{passive:false});

function upd() {
  if (g.s!=='play') return;
  g.fr++; g.dist+=g.spd; g.score+=g.spd*0.02; g.rf+=0.28;
  g.sx=(g.sx+g.spd)%100; g.mx=(g.mx+g.spd)%130;
  if(g.cd>0) g.cd--;
  if(g.hitFlash>0) g.hitFlash--;
  g.ry+=(LY[g.lane]-g.ry)*0.22;
  g.stam-=0.042;
  if(g.stam<=0){g.s='over';return;}
  g.spd=Math.min(13, 4+g.hoffs*0.5+g.dist*0.0004);

  if(g.dist>g.nextObj){
    g.objs.push({t:Math.random()<0.55?'p':'c', lane:Math.floor(Math.random()*3), x:W+30});
    g.nextObj=g.dist+140+Math.random()*240;
  }
  if(!g.hz && g.dist>g.nextHz){
    g.hz={x:W+30, w:210, tl:Math.floor(Math.random()*3), ok:false, flash:0};
    g.nextHz=g.dist+1500+Math.random()*700;
  }

  for(let i=g.objs.length-1;i>=0;i--){
    const o=g.objs[i];
    o.x-=g.spd;
    if(o.x<-40){g.objs.splice(i,1);continue;}
    if(Math.abs(o.x-150)<26 && Math.abs(LY[o.lane]-g.ry)<25){
      if(o.t==='p'){g.score+=15;g.stam=Math.min(100,g.stam+18);}
      else{g.stam-=22;g.hitFlash=18;if(g.stam<=0){g.s='over';return;}}
      g.objs.splice(i,1);
    }
  }

  if(g.hz){
    g.hz.x-=g.spd;
    const h=g.hz;
    if(h.flash>0) h.flash--;
    const inside=150>=h.x&&150<=h.x+h.w;
    if(inside&&!h.ok&&g.lane===h.tl){
      h.ok=true; h.flash=60;
      g.hoffs++; g.score+=75; g.stam=100;
    }
    if(!h.ok && h.x+h.w<140){g.s='over';return;}
    if(h.x<-400) g.hz=null;
  }
}

// ── Helpers ───────────────────────────────────────────────────────────────────
function rr(x,y,w,h,r){ctx.beginPath();ctx.roundRect(x,y,w,h,r);}

function drawRunner(x,y,f){
  const leg=Math.sin(f)*16, arm=Math.sin(f+Math.PI)*12, bob=Math.abs(Math.sin(f))*2;
  ctx.save(); ctx.translate(x,y+bob);
  ctx.fillStyle='rgba(0,0,0,0.18)';
  ctx.beginPath();ctx.ellipse(0,28,14,4,0,0,Math.PI*2);ctx.fill();
  ctx.strokeStyle='#333';ctx.lineWidth=5;ctx.lineCap='round';
  ctx.beginPath();ctx.moveTo(0,8);ctx.lineTo(-leg*.5,18);ctx.lineTo(-leg,28);ctx.stroke();
  ctx.beginPath();ctx.moveTo(0,8);ctx.lineTo(leg*.5,18);ctx.lineTo(leg,28);ctx.stroke();
  ctx.fillStyle=R; rr(-9,-10,18,18,3); ctx.fill();
  ctx.fillStyle=W2;ctx.font='bold 7px Arial';ctx.textAlign='center';ctx.fillText('NC',0,1);
  ctx.strokeStyle='#FDBCB4';ctx.lineWidth=4;
  ctx.beginPath();ctx.moveTo(-7,-6);ctx.lineTo(-7-arm,6);ctx.stroke();
  ctx.beginPath();ctx.moveTo(7,-6);ctx.lineTo(7+arm,6);ctx.stroke();
  ctx.fillStyle='#FDBCB4';ctx.beginPath();ctx.arc(0,-19,10,0,Math.PI*2);ctx.fill();
  ctx.fillStyle=R;ctx.beginPath();ctx.arc(0,-26,9,Math.PI,0);ctx.fill();
  ctx.fillRect(-13,-29,26,6);
  ctx.strokeStyle=GD;ctx.lineWidth=4;ctx.lineCap='round';
  ctx.beginPath();ctx.moveTo(8+arm,4);ctx.lineTo(22+arm,-1);ctx.stroke();
  ctx.restore();
}

function drawPrize(x,y){
  ctx.save();ctx.translate(x,y);
  ctx.fillStyle=BL; rr(-7,-14,14,24,3);ctx.fill();
  ctx.fillStyle='#9ef';ctx.fillRect(-4,-12,3,8);
  ctx.fillStyle='#ddd'; rr(-4,-18,8,6,2);ctx.fill();
  ctx.fillStyle=GD;ctx.font='11px Arial';ctx.textAlign='center';ctx.fillText('★',13,-12);
  ctx.restore();
}

function drawCone(x,y){
  ctx.save();ctx.translate(x,y);
  ctx.fillStyle=OR;
  ctx.beginPath();ctx.moveTo(0,-18);ctx.lineTo(13,14);ctx.lineTo(-13,14);ctx.closePath();ctx.fill();
  ctx.strokeStyle=W2;ctx.lineWidth=2;
  ctx.beginPath();ctx.moveTo(-7,0);ctx.lineTo(7,0);ctx.stroke();
  ctx.fillStyle='#c44000';ctx.fillRect(-14,12,28,5);
  ctx.restore();
}

function drawBg(){
  const sky=ctx.createLinearGradient(0,0,0,TY);
  sky.addColorStop(0,'#5fa8d3');sky.addColorStop(1,'#aad8f0');
  ctx.fillStyle=sky;ctx.fillRect(0,0,W,TY);

  // clouds
  ctx.fillStyle='rgba(255,255,255,0.75)';
  [[110,38,32],[310,22,25],[530,32,28],[700,18,20]].forEach(([cx,cy,r])=>{
    ctx.beginPath();ctx.arc(cx,cy,r,0,Math.PI*2);ctx.fill();
    ctx.beginPath();ctx.arc(cx+r*.7,cy+5,r*.65,0,Math.PI*2);ctx.fill();
    ctx.beginPath();ctx.arc(cx-r*.6,cy+6,r*.55,0,Math.PI*2);ctx.fill();
  });

  ctx.fillStyle='#3a7a23';ctx.fillRect(0,TY+TH,W,H-TY-TH);

  const ts=ctx.createLinearGradient(0,TY,0,TY+TH);
  ts.addColorStop(0,'#d4a96a');ts.addColorStop(.5,'#c8955a');ts.addColorStop(1,'#d4a96a');
  ctx.fillStyle=ts;ctx.fillRect(0,TY,W,TH);

  for(let x=-g.sx;x<W+100;x+=100){
    ctx.fillStyle=R;ctx.fillRect(x,TY-10,50,10);ctx.fillStyle=W2;ctx.fillRect(x+50,TY-10,50,10);
    ctx.fillStyle=R;ctx.fillRect(x,TY+TH,50,10);ctx.fillStyle=W2;ctx.fillRect(x+50,TY+TH,50,10);
  }
  ctx.strokeStyle=W2;ctx.lineWidth=3;
  ctx.beginPath();ctx.moveTo(0,TY);ctx.lineTo(W,TY);ctx.stroke();
  ctx.beginPath();ctx.moveTo(0,TY+TH);ctx.lineTo(W,TY+TH);ctx.stroke();

  ctx.setLineDash([38,22]);ctx.strokeStyle='rgba(255,255,255,0.5)';ctx.lineWidth=2;
  for(let l=1;l<3;l++){
    ctx.beginPath();ctx.moveTo(-g.mx,TY+l*LH);ctx.lineTo(W+130,TY+l*LH);ctx.stroke();
  }
  ctx.setLineDash([]);
}

function drawHz(){
  const h=g.hz; if(!h||h.x>W) return;
  const{x,w,tl,ok,flash}=h;
  for(let l=0;l<3;l++){
    const ly=TY+l*LH;
    if(l===tl){
      ctx.fillStyle=ok
        ?(flash%8<4?'rgba(80,255,80,0.75)':'rgba(255,255,80,0.75)')
        :'rgba(255,215,0,0.42)';
    } else {
      ctx.fillStyle='rgba(100,140,255,0.18)';
    }
    ctx.fillRect(x,ly,w,LH);
  }
  // chevrons on target lane
  if(!ok){
    ctx.save();
    ctx.beginPath();ctx.rect(x,TY+tl*LH,w,LH);ctx.clip();
    ctx.strokeStyle='rgba(255,215,0,0.45)';ctx.lineWidth=6;
    for(let cx=x-20;cx<x+w+40;cx+=28){
      ctx.beginPath();ctx.moveTo(cx,TY+tl*LH);ctx.lineTo(cx+14,TY+(tl+1)*LH);ctx.stroke();
    }
    ctx.restore();
  }
  ctx.strokeStyle=ok?'#44ff88':GD;ctx.lineWidth=3;
  ctx.strokeRect(x,TY,w,TH);
  const lx=Math.min(x+w/2,W-65);
  ctx.fillStyle=W2;ctx.font='bold 12px Arial';ctx.textAlign='center';
  ctx.fillText('HANDOFF',lx,TY-16);
  ctx.fillText('ZONE',lx,TY-4);
  if(!ok){
    ctx.fillStyle=GD;ctx.font='22px Arial';
    ctx.fillText('◉',lx,TY+tl*LH+LH/2+8);
  }
  if(ok&&flash>0){
    ctx.fillStyle=GD;ctx.font='bold 22px Arial';ctx.textAlign='center';
    ctx.fillText('✓ NICE PASS!',x+w/2,TY+TH/2+8);
  }
}

function drawHUD(){
  // hit flash overlay
  if(g.hitFlash>0){
    ctx.fillStyle=`rgba(200,0,0,${g.hitFlash*0.03})`;
    ctx.fillRect(0,0,W,H);
  }

  // score panel
  ctx.fillStyle='rgba(0,0,0,0.62)'; rr(10,8,178,72,8); ctx.fill();
  ctx.fillStyle=W2;ctx.font='bold 15px Arial';ctx.textAlign='left';
  ctx.fillText(`Score: ${Math.floor(g.score)}`,18,30);

  ctx.fillStyle='rgba(0,0,0,0.5)'; rr(18,38,148,15,5); ctx.fill();
  const sc=g.stam>50?'#44cc44':g.stam>25?'#ffaa00':R;
  ctx.fillStyle=sc; rr(18,38,g.stam*1.48,15,5); ctx.fill();
  ctx.fillStyle=W2;ctx.font='bold 10px Arial';ctx.fillText('STAMINA',22,50);

  ctx.fillStyle=GD;ctx.font='13px Arial';
  ctx.fillText(`🏃 ${g.hoffs} handoff${g.hoffs!==1?'s':''}`,18,72);

  // speed / dist top right
  ctx.fillStyle='rgba(0,0,0,0.62)'; rr(W-125,8,115,40,8); ctx.fill();
  ctx.fillStyle=W2;ctx.font='bold 13px Arial';ctx.textAlign='right';
  ctx.fillText(`⚡ ${g.spd.toFixed(1)}`,W-18,28);
  ctx.fillStyle='rgba(255,255,255,0.5)';ctx.font='12px Arial';
  ctx.fillText(`${Math.floor(g.dist)} m`,W-18,46);

  // lane panel for upcoming handoff
  if(g.hz){
    const h=g.hz, px=W-80, py=H-118;
    ctx.fillStyle='rgba(0,0,0,0.68)'; rr(px-10,py-8,74,108,8); ctx.fill();
    ctx.fillStyle=W2;ctx.font='bold 10px Arial';ctx.textAlign='center';
    ctx.fillText('HANDOFF',px+27,py+8);
    ctx.fillText('LANE:',px+27,py+20);
    for(let l=0;l<3;l++){
      const by=py+28+l*26;
      if(l===h.tl){
        ctx.fillStyle=h.ok?'#44cc44':(Math.sin(g.fr*.15)>0?GD:'#bb9900');
        rr(px+4,by,46,21,4);ctx.fill();
        ctx.fillStyle=BK;ctx.font='bold 10px Arial';ctx.textAlign='center';
        ctx.fillText(h.ok?'✓ DONE':'← HERE',px+27,by+14);
      } else {
        ctx.fillStyle='rgba(255,255,255,0.12)'; rr(px+4,by,46,21,4); ctx.fill();
      }
    }
  }

  // controls hint
  ctx.fillStyle='rgba(255,255,255,0.35)';ctx.font='11px Arial';ctx.textAlign='left';
  ctx.fillText('↑/W · ↓/S  change lanes',12,H-10);
}

function drawStart(){
  ctx.fillStyle='rgba(0,0,0,0.8)';ctx.fillRect(0,0,W,H);
  ctx.font='64px Arial';ctx.textAlign='center';ctx.fillText('🐺',W/2,H/2-92);
  ctx.fillStyle=R;ctx.font='bold 36px Arial';ctx.fillText('NC STATE RUN CLUB',W/2,H/2-42);
  ctx.fillStyle=W2;ctx.font='20px Arial';ctx.fillText('Track Relay Runner',W/2,H/2+0);
  ctx.fillStyle='rgba(255,255,255,0.6)';ctx.font='14px Arial';
  ctx.fillText('Collect water bottles  •  Dodge cones  •  Pass the baton!',W/2,H/2+26);
  if(Math.floor(Date.now()/550)%2){
    ctx.fillStyle=GD;ctx.font='bold 17px Arial';
    ctx.fillText('Press any key or tap to start',W/2,H/2+58);
  }
  ctx.fillStyle='rgba(255,255,255,0.4)';ctx.font='13px Arial';
  ctx.fillText('↑ / W = Lane Up       ↓ / S = Lane Down',W/2,H/2+86);
  ctx.fillStyle=R;ctx.font='bold 15px Arial';
  ctx.fillText('Go Pack!  🐾',W/2,H/2+112);
}

function drawOver(){
  ctx.fillStyle='rgba(0,0,0,0.84)';ctx.fillRect(0,0,W,H);
  ctx.fillStyle=R;ctx.font='bold 52px Arial';ctx.textAlign='center';ctx.fillText('RACE OVER!',W/2,H/2-82);
  ctx.fillStyle=GD;ctx.font='bold 26px Arial';ctx.fillText(`Score: ${Math.floor(g.score)}`,W/2,H/2-35);
  ctx.fillStyle=W2;ctx.font='22px Arial';
  ctx.fillText(`Handoffs completed: ${g.hoffs}`,W/2,H/2+5);
  ctx.fillText(`Distance: ${Math.floor(g.dist)} m`,W/2,H/2+35);
  if(Math.floor(Date.now()/550)%2){
    ctx.fillStyle=GD;ctx.font='bold 17px Arial';
    ctx.fillText('Press any key or tap to try again',W/2,H/2+75);
  }
  ctx.fillStyle='rgba(255,255,255,0.4)';ctx.font='13px Arial';
  ctx.fillText('Go Pack!  🐾',W/2,H/2+106);
}

function draw(){
  ctx.clearRect(0,0,W,H);
  drawBg();
  for(const o of g.objs) o.t==='p'?drawPrize(o.x,LY[o.lane]):drawCone(o.x,LY[o.lane]);
  drawHz();
  drawRunner(150,g.ry,g.rf);
  drawHUD();
  if(g.s==='start') drawStart();
  if(g.s==='over') drawOver();
}

function loop(){ upd(); draw(); requestAnimationFrame(loop); }
reset(); loop();
</script>
</body>
</html>
