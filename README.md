<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Mini Studio v2.1 – Browser 3D Editor</title>
<style>
:root{--bg:#232326;--p:#2d2d31;--p2:#38383d;--ln:#141416;--tx:#d9d9de;--mu:#8d8d96;--ac:#e8883a}
*{box-sizing:border-box}
html,body{height:100%;margin:0}
body{background:var(--bg);color:var(--tx);font:13px system-ui,sans-serif;
  height:100vh;height:100dvh;overflow:hidden;
  display:grid;
  grid-template-columns:190px minmax(0,1fr) 260px;
  grid-template-rows:34px minmax(0,1fr) 22px;
  grid-template-areas:"hd hd hd" "ol vp pr" "st st st"}
header{grid-area:hd;background:var(--p);border-bottom:1px solid var(--ln);display:flex;gap:4px;align-items:center;padding:0 8px;overflow-x:auto;white-space:nowrap}
header b{color:var(--ac);margin-right:8px}header i{width:1px;height:18px;background:var(--ln);margin:0 4px}
button,select,input{font:inherit;color:var(--tx);background:var(--p2);border:1px solid var(--ln);border-radius:4px;padding:3px 8px}
button{cursor:pointer}button:hover{border-color:var(--ac)}button.on{background:var(--ac);color:#111;border-color:var(--ac)}
#outl,#props{background:var(--p);overflow-y:auto;padding:8px;min-height:0}
#outl{grid-area:ol;border-right:1px solid var(--ln)}
#props{grid-area:pr;border-left:1px solid var(--ln)}
h3{margin:2px 0 8px;font-size:11px;letter-spacing:.08em;text-transform:uppercase;color:var(--mu)}
.it{display:flex;align-items:center;gap:6px;padding:3px 6px;border-radius:4px;cursor:pointer}
.it:hover{background:var(--p2)}.it.s{background:var(--ac);color:#111}.it span{flex:1;overflow:hidden;text-overflow:ellipsis}
.it u{text-decoration:none;opacity:.7}
#vp{grid-area:vp;position:relative;min-width:0;min-height:0;overflow:hidden}
#vp canvas{position:absolute;inset:0;width:100%;height:100%;display:block;touch-action:none}
#st{grid-area:st;background:var(--p);border-top:1px solid var(--ln);color:var(--mu);padding:3px 10px;font-size:11px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.r3{display:grid;grid-template-columns:repeat(3,1fr);gap:3px;margin-bottom:6px}.r3 input{width:100%;min-width:0;padding:3px 4px}
.fr{display:flex;align-items:center;gap:6px;margin:6px 0}.fr label{width:75px;color:var(--mu);flex-shrink:0}.fr input[type=range]{flex:1;padding:0}
.lbl{color:var(--mu);font-size:11px;margin:8px 0 3px}
#np{color:var(--mu);margin:0}

@media(max-width:600px){
  body{height:auto;min-height:100dvh;overflow:auto;
    grid-template-columns:minmax(0,1fr);
    grid-template-rows:34px 60vh auto auto 22px;
    grid-template-areas:"hd" "vp" "ol" "pr" "st"}
  #outl,#props{border:0;border-top:1px solid var(--ln);overflow:visible}
}
</style>
</head>
<body>
<header>
 <b>MiniStudio v2.1</b>
 <select id="add">
   <option value="">+ Add…</option>
   <option>Cube</option>
   <option>Sphere</option>
   <option>Cylinder</option>
   <option>Cone</option>
   <option>Torus</option>
   <option>Plane</option>
   <option>Icosphere</option>
   <option>PointLight</option>
   <option>SpotLight</option>
   <option>DirLight</option>
 </select>
 <i></i>
 <button id="mT" class="on" title="G">Move</button><button id="mR" title="R">Rotate</button><button id="mS" title="S">Scale</button>
 <button id="snap">Snap</button><button id="loc">Local</button>
 <i></i>
 <button id="und">Undo</button><button id="red">Redo</button><button id="dup">Duplicate</button><button id="del">Delete</button>
 <i></i>
 <select id="shade"><option value="solid">Solid</option><option value="wire">Wireframe</option></select>
 <button id="save">Save .json</button><button id="load">Load</button><button id="glb">Export .glb</button><button id="imp">Import .glb</button>
 <input id="file" type="file" hidden>
</header>
<div id="outl"><h3>Outliner</h3><div id="list"></div></div>
<div id="vp"></div>
<div id="props"><h3>Properties</h3>
 <p id="np">Nothing selected.<br>Click an object or add one from “+ Add…”.</p>
 <div id="pp" hidden>
  <div class="fr"><label>Name</label><input id="nm" style="flex:1;min-width:0"></div>
  <div class="lbl">Location</div><div class="r3" id="pos"></div>
  <div class="lbl">Rotation (°)</div><div class="r3" id="rot"></div>
  <div class="lbl">Scale</div><div class="r3" id="scl"></div>

  <div id="mat-props">
    <div class="lbl">Material</div>
    <div class="fr"><label>Colour</label><input id="col" type="color"></div>
    <div class="fr"><label>Roughness</label><input id="rgh" type="range" min="0" max="1" step=".01"></div>
    <div class="fr"><label>Metalness</label><input id="met" type="range" min="0" max="1" step=".01"></div>
    <div class="fr"><label>Wireframe</label><input id="wr" type="checkbox"></div>
    <div class="fr">
      <label>Texture</label>
      <button id="tex-btn" style="flex:1;min-width:0">Upload…</button>
      <button id="tex-del" title="Remove texture" style="padding:3px 7px">×</button>
      <input id="tex-file" type="file" accept="image/*" hidden>
    </div>
  </div>

  <div id="light-props" hidden>
    <div class="lbl">Light Settings</div>
    <div class="fr"><label>Colour</label><input id="lcol" type="color"></div>
    <div class="fr"><label>Intensity</label><input id="li-int" type="range" min="0" max="10" step=".1"></div>
    <div class="fr" id="li-angle-row"><label>Cone Angle</label><input id="li-ang" type="range" min="0.1" max="1.5" step=".05"></div>
  </div>
 </div>
</div>
<div id="st">Orbit: left-drag · Pan: right-drag · Zoom: wheel · G/R/S transform · Shift+D duplicate · X delete · F focus · 1/3/7 views · Ctrl+Z undo</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/TransformControls.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/GLTFLoader.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/exporters/GLTFExporter.js"></script>
<script>
(function(){
var $=function(i){return document.getElementById(i)},vp=$('vp');
var renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2));renderer.shadowMap.enabled=true;
renderer.outputEncoding=THREE.sRGBEncoding;renderer.toneMapping=THREE.ACESFilmicToneMapping;
vp.appendChild(renderer.domElement);
var scene=new THREE.Scene();scene.background=new THREE.Color(0x3a3a40);
var cam=new THREE.PerspectiveCamera(50,1,.1,500);cam.position.set(6,5,8);
var orbit=new THREE.OrbitControls(cam,renderer.domElement);orbit.enableDamping=true;orbit.dampingFactor=.12;
scene.add(new THREE.HemisphereLight(0xffffff,0x444455,.4));
var sun=new THREE.DirectionalLight(0xffffff,.8);sun.position.set(6,10,5);sun.castShadow=true;
sun.shadow.mapSize.set(2048,2048);['left','bottom'].forEach(function(k){sun.shadow.camera[k]=-12});['right','top'].forEach(function(k){sun.shadow.camera[k]=12});scene.add(sun);
var grid=new THREE.GridHelper(40,40,0x777788,0x55555f);scene.add(grid);
var floor=new THREE.Mesh(new THREE.PlaneGeometry(40,40),new THREE.ShadowMaterial({opacity:.25}));floor.rotation.x=-Math.PI/2;floor.receiveShadow=true;scene.add(floor);
var world=new THREE.Group();scene.add(world);
var tc=new THREE.TransformControls(cam,renderer.domElement);scene.add(tc);
var hl=new THREE.BoxHelper(new THREE.Mesh(new THREE.BoxGeometry()),0xe8883a);hl.visible=false;scene.add(hl);
var sel=null,counter={};
var texLoader=new THREE.TextureLoader();

/* ---- primitives ---- */
var GEO={Cube:function(){return new THREE.BoxGeometry(1.4,1.4,1.4)},Sphere:function(){return new THREE.SphereGeometry(.9,64,48)},
 Cylinder:function(){return new THREE.CylinderGeometry(.7,.7,1.6,64)},Cone:function(){return new THREE.ConeGeometry(.9,1.8,64)},
 Torus:function(){return new THREE.TorusGeometry(.8,.3,32,96)},Plane:function(){return new THREE.PlaneGeometry(2.5,2.5)},
 Icosphere:function(){return new THREE.IcosahedronGeometry(.95,2)},Light:function(){return new THREE.SphereGeometry(.14,16,12)}};

function make(type,name){
 var m,L;
 var isLight=['PointLight','SpotLight','DirLight','Light'].includes(type);
 if(isLight){
  var ptype=type==='Light'?'PointLight':type;
  m=new THREE.Mesh(GEO.Light(),new THREE.MeshBasicMaterial({color:0xfff2cc,wireframe:true}));
  if(ptype==='SpotLight'){
   L=new THREE.SpotLight(0xfff2cc,2,25,Math.PI/4,.3,1);
   L.castShadow=true;L.shadow.mapSize.set(1024,1024);
   m.add(L);m.add(L.target);L.target.position.set(0,-2,0);
  }else if(ptype==='DirLight'){
   L=new THREE.DirectionalLight(0xfff2cc,1.5);
   L.castShadow=true;L.shadow.mapSize.set(1024,1024);
   m.add(L);m.add(L.target);L.target.position.set(0,-2,0);
  }else{
   L=new THREE.PointLight(0xfff2cc,1.6,0,2);
   L.castShadow=true;m.add(L);
  }
  m.userData.light=L;
 }else{
  m=new THREE.Mesh(GEO[type](),new THREE.MeshStandardMaterial({color:0xcccccc,roughness:.5,metalness:.1,side:type==='Plane'?THREE.DoubleSide:THREE.FrontSide}));
  m.castShadow=m.receiveShadow=true;
  if(type==='Plane'){m.rotation.x=-Math.PI/2}
 }
 counter[type]=(counter[type]||0)+1;
 m.name=name||(type+(counter[type]>1?'.'+String(counter[type]-1).padStart(3,'0'):''));
 m.userData.prim=type;world.add(m);return m;
}

function addObj(type){
 var m=make(type);
 if(['PointLight','SpotLight','DirLight','Light'].includes(type))m.position.set(2,3,2);
 else m.position.y=type==='Plane'?0.001:.8;
 snapshot();select(m);
}

/* ---- selection / outliner ---- */
function rootOf(o){while(o&&o.parent!==world)o=o.parent;return o}
function select(o){sel=o||null;tc.detach();if(sel)tc.attach(sel);ui()}
function ui(){
 var l=$('list');l.innerHTML='';
 world.children.forEach(function(o){
  var d=document.createElement('div');d.className='it'+(o===sel?' s':'');
  var s=document.createElement('span');s.textContent=o.name;d.appendChild(s);
  var e=document.createElement('u');e.textContent=o.visible?'👁':'—';e.title='Show / hide';
  e.onclick=function(ev){ev.stopPropagation();o.visible=!o.visible;if(o===sel&&!o.visible)select(null);else ui();snapshot()};d.appendChild(e);
  d.onclick=function(){select(o)};d.ondblclick=function(){var n=prompt('Rename',o.name);if(n){o.name=n;ui();snapshot()}};
  l.appendChild(d);
 });
 $('np').hidden=!!sel;$('pp').hidden=!sel;
 if(sel)sync();
}

function mat(o){return o&&o.material&&o.material.isMeshStandardMaterial?o.material:null}

function sync(){
 $('nm').value=sel.name;
 ['pos','rot','scl'].forEach(function(k,i){
  var v=i===0?sel.position:i===1?new THREE.Vector3(sel.rotation.x*57.2958,sel.rotation.y*57.2958,sel.rotation.z*57.2958):sel.scale;
  [v.x,v.y,v.z].forEach(function(n,j){$(k).children[j].value=(+n).toFixed(2)})
 });
 var M=mat(sel),L=sel.userData.light;
 $('mat-props').hidden=!M;
 $('light-props').hidden=!L;

 if(M){
  $('col').value='#'+M.color.getHexString();
  $('rgh').value=M.roughness;$('met').value=M.metalness;$('wr').checked=M.wireframe;
  $('tex-btn').textContent=M.map?'Replace Image':'Upload…';
 }else if(L){
  $('lcol').value='#'+L.color.getHexString();
  $('li-int').value=L.intensity;
  if(L.isSpotLight){
   $('li-angle-row').hidden=false;
   $('li-ang').value=L.angle;
  }else{
   $('li-angle-row').hidden=true;
  }
 }
}

['pos','rot','scl'].forEach(function(k){
 for(var j=0;j<3;j++){var i=document.createElement('input');i.type='number';i.step='.1';
  i.onchange=function(){
   if(!sel)return;var c=$(k).children,v=[+c[0].value,+c[1].value,+c[2].value];
   if(k==='pos')sel.position.set(v[0],v[1],v[2]);
   else if(k==='rot')sel.rotation.set(v[0]/57.2958,v[1]/57.2958,v[2]/57.2958);
   else sel.scale.set(v[0]||.01,v[1]||.01,v[2]||.01);
   snapshot()};
  $(k).appendChild(i)}
});

$('nm').onchange=function(){if(sel){sel.name=this.value;ui();snapshot()}};

/* Material events */
$('col').oninput=function(){if(sel&&mat(sel)){mat(sel).color.set(this.value)}};
$('col').onchange=snapshot;
$('rgh').oninput=function(){if(sel&&mat(sel))mat(sel).roughness=+this.value};$('rgh').onchange=snapshot;
$('met').oninput=function(){if(sel&&mat(sel))mat(sel).metalness=+this.value};$('met').onchange=snapshot;
$('wr').onchange=function(){if(sel&&mat(sel)){mat(sel).wireframe=this.checked;snapshot()}};

/* Texture upload & clear */
$('tex-btn').onclick=function(){$('tex-file').click()};
$('tex-file').onchange=function(){
 var f=this.files[0];this.value='';
 if(!f||!sel||!mat(sel))return;
 var rd=new FileReader();
 rd.onload=function(e){
  var imgData=e.target.result;
  texLoader.load(imgData,function(tex){
   tex.encoding=THREE.sRGBEncoding;
   var M=mat(sel);
   if(!M)return;
   if(M.map)M.map.dispose();
   M.map=tex;M.map.userData={src:imgData};M.needsUpdate=true;
   sync();snapshot();
  });
 };
 rd.readAsDataURL(f);
};
$('tex-del').onclick=function(){
 if(!sel||!mat(sel))return;
 var M=mat(sel);
 if(M.map){M.map.dispose();M.map=null;M.needsUpdate=true;sync();snapshot()}
};

/* Light events */
$('lcol').oninput=function(){
 if(sel&&sel.userData.light){
  sel.userData.light.color.set(this.value);
  sel.material.color.set(this.value);
 }
};
$('lcol').onchange=snapshot;
$('li-int').oninput=function(){if(sel&&sel.userData.light)sel.userData.light.intensity=+this.value};
$('li-int').onchange=snapshot;
$('li-ang').oninput=function(){if(sel&&sel.userData.light&&sel.userData.light.isSpotLight)sel.userData.light.angle=+this.value};
$('li-ang').onchange=snapshot;

/* ---- history (undo/redo) + autosave ---- */
var hist=[],hi=-1;
function dump(){
 return JSON.stringify({sel:world.children.indexOf(sel),o:world.children.filter(function(o){return o.userData.prim}).map(function(o){
  var M=mat(o),L=o.userData.light;
  return{t:o.userData.prim,n:o.name,p:o.position.toArray(),q:o.quaternion.toArray(),s:o.scale.toArray(),v:o.visible,
   c:'#'+(M?M.color:L?L.color:new THREE.Color()).getHexString(),r:M?M.roughness:0,m:M?M.metalness:0,w:M?M.wireframe:false,
   tex:M&&M.map&&M.map.userData&&M.map.userData.src?M.map.userData.src:null,
   lint:L?L.intensity:null,lang:L&&L.isSpotLight?L.angle:null}})});
}

function load(txt){
 var d=JSON.parse(txt);
 world.children.filter(function(o){return o.userData.prim}).forEach(function(o){world.remove(o)});
 counter={};
 d.o.forEach(function(r){
  var m=make(r.t,r.n);m.position.fromArray(r.p);m.quaternion.fromArray(r.q);m.scale.fromArray(r.s);m.visible=r.v;
  var M=mat(m),L=m.userData.light;
  if(M){
   M.color.set(r.c);M.roughness=r.r;M.metalness=r.m;M.wireframe=r.w;
   if(r.tex){
    (function(targetMat,imgSrc){
     texLoader.load(imgSrc,function(tex){
      tex.encoding=THREE.sRGBEncoding;targetMat.map=tex;targetMat.map.userData={src:imgSrc};targetMat.needsUpdate=true;if(sel===m)sync();
     });
    })(M,r.tex);
   }
  }else if(L){
   m.material.color.set(r.c);L.color.set(r.c);
   if(r.lint!==null)L.intensity=r.lint;
   if(r.lang!==null&&L.isSpotLight)L.angle=r.lang;
  }
 });
 select(world.children[d.sel]||null);
}

function snapshot(){
 var s=dump();hist=hist.slice(0,hi+1);hist.push(s);if(hist.length>80)hist.shift();hi=hist.length-1;
 try{localStorage.setItem('ministudio_v21',s)}catch(e){}ui();
}
function undo(){if(hi>0){hi--;load(hist[hi])}}
function redo(){if(hi<hist.length-1){hi++;load(hist[hi])}}

/* ---- actions ---- */
function del(){if(!sel)return;world.remove(sel);select(null);snapshot()}
function dup(){
 if(!sel)return;var c=sel.clone();
 if(c.material&&c.material.clone)c.material=sel.material.clone();
 if(sel.userData.light){
  c.remove(c.children[0]);
  var L=sel.userData.light.clone();
  c.add(L);c.userData.light=L;
  if(L.target){c.add(L.target);L.target.position.set(0,-2,0)}
 }
 c.name=sel.name+'.copy';c.position.x+=.6;c.position.z+=.6;world.add(c);snapshot();select(c);
}

function dl(name,blob){var a=document.createElement('a');a.href=URL.createObjectURL(blob);a.download=name;a.click();setTimeout(function(){URL.revokeObjectURL(a.href)},2000)}
$('add').onchange=function(){if(this.value)addObj(this.value);this.value=''};
$('del').onclick=del;$('dup').onclick=dup;$('und').onclick=undo;$('red').onclick=redo;
$('save').onclick=function(){dl('scene.json',new Blob([dump()],{type:'application/json'}))};
$('glb').onclick=function(){
 tc.detach();hl.visible=false;
 new THREE.GLTFExporter().parse(world,function(b){dl('scene.glb',new Blob([b],{type:'model/gltf-binary'}));if(sel)tc.attach(sel)},{binary:true});
};

var mode=null;
$('load').onclick=function(){mode='json';$('file').accept='.json';$('file').click()};
$('imp').onclick=function(){mode='glb';$('file').accept='.glb,.gltf';$('file').click()};
$('file').onchange=function(){
 var f=this.files[0];this.value='';if(!f)return;var rd=new FileReader();
 if(mode==='json'){rd.onload=function(){try{load(rd.result);snapshot()}catch(e){alert('Not a valid scene file.')}};rd.readAsText(f)}
 else{rd.onload=function(){new THREE.GLTFLoader().parse(rd.result,'',function(g){
   var o=g.scene;o.name=f.name.replace(/\.\w+$/,'');o.traverse(function(c){if(c.isMesh)c.castShadow=c.receiveShadow=true});
   world.add(o);select(o);ui()},function(){alert('Could not read that model.')})};rd.readAsArrayBuffer(f)}
};

function setMode(m){tc.setMode(m);[['mT','translate'],['mR','rotate'],['mS','scale']].forEach(function(p){$(p[0]).className=p[1]===m?'on':''})}
$('mT').onclick=function(){setMode('translate')};$('mR').onclick=function(){setMode('rotate')};$('mS').onclick=function(){setMode('scale')};
$('snap').onclick=function(){var on=this.className!=='on';this.className=on?'on':'';tc.setTranslationSnap(on?.5:null);tc.setRotationSnap(on?Math.PI/12:null);tc.setScaleSnap(on?.1:null)};
$('loc').onclick=function(){var on=this.className!=='on';this.className=on?'on':'';tc.setSpace(on?'local':'world')};
$('shade').onchange=function(){var w=this.value==='wire';world.traverse(function(o){var M=o.material;if(M&&M.isMeshStandardMaterial)M.wireframe=w});if(sel)sync()};
tc.addEventListener('dragging-changed',function(e){orbit.enabled=!e.value;if(!e.value)snapshot()});
tc.addEventListener('objectChange',function(){if(sel)sync()});

function view(x,y,z){var t=orbit.target,d=cam.position.distanceTo(t);cam.position.set(t.x+x*d,t.y+y*d,t.z+z*d)}
addEventListener('keydown',function(e){
 if(/INPUT|SELECT|TEXTAREA/.test(e.target.tagName))return;
 var k=e.key.toLowerCase();
 if((e.ctrlKey||e.metaKey)&&k==='z'){e.preventDefault();e.shiftKey?redo():undo();return}
 if((e.ctrlKey||e.metaKey)&&k==='y'){e.preventDefault();redo();return}
 if(k==='g')setMode('translate');else if(k==='r')setMode('rotate');else if(k==='s')setMode('scale');
 else if(k==='x'||k==='delete'||k==='backspace')del();
 else if(k==='d'&&e.shiftKey)dup();
 else if(k==='f'&&sel){orbit.target.copy(sel.getWorldPosition(new THREE.Vector3()))}
 else if(k==='1')view(0,0,1);else if(k==='3')view(1,0,0);else if(k==='7')view(0,1,.001);
});

/* ---- picking ---- */
var ray=new THREE.Raycaster(),ms=new THREE.Vector2(),dn=null;
renderer.domElement.addEventListener('pointerdown',function(e){dn=[e.clientX,e.clientY]});
renderer.domElement.addEventListener('pointerup',function(e){
 if(!dn||Math.abs(e.clientX-dn[0])+Math.abs(e.clientY-dn[1])>4||tc.dragging||e.button!==0)return;
 var r=renderer.domElement.getBoundingClientRect();
 ms.set((e.clientX-r.left)/r.width*2-1,-(e.clientY-r.top)/r.height*2+1);ray.setFromCamera(ms,cam);
 var h=ray.intersectObjects(world.children.filter(function(o){return o.visible}),true)[0];
 select(h?rootOf(h.object):null);
});

/* ---- resize ---- */
function resize(){
 var w=vp.clientWidth,h=vp.clientHeight;
 if(!w||!h)return;
 renderer.setSize(w,h,false);
 cam.aspect=w/h;cam.updateProjectionMatrix();
}
addEventListener('resize',resize);
if(window.ResizeObserver)new ResizeObserver(resize).observe(vp);
resize();

/* ---- boot ---- */
var saved=null;try{saved=localStorage.getItem('ministudio_v21')}catch(e){}
if(saved){try{load(saved)}catch(e){saved=null}}
if(!saved||!world.children.length){
 var a=make('Cube');a.position.set(-2,.7,0);a.material.color.set('#e8883a');
 var b=make('Sphere');b.position.set(0,.9,0);b.material.color.set('#4aa3ff');b.material.metalness=.8;b.material.roughness=.2;
 var c=make('Torus');c.position.set(2.2,.9,0);c.rotation.x=1.2;c.material.color.set('#7bd88f');
 var l=make('PointLight');l.position.set(0,3.2,2);
}
snapshot();select(null);
(function loop(){
 requestAnimationFrame(loop);orbit.update();
 if(sel&&sel.visible){hl.visible=true;hl.setFromObject(sel)}else hl.visible=false;
 renderer.render(scene,cam);
})();
})();
</script>
</body>
</html>
