
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Supermercado Simulator 3D</title>
<style>
body{margin:0;overflow:hidden;font-family:Arial}
#hud{position:fixed;top:12px;left:12px;background:#10251eeb;color:white;padding:16px;border-radius:12px;z-index:2}
#hud h2{margin:0 0 10px;color:#71ff9b}
#msg{position:fixed;bottom:25px;left:50%;transform:translateX(-50%);background:#10251eeb;color:white;padding:15px;border-radius:10px;text-align:center;z-index:2}
#start{position:fixed;inset:0;background:#14251f;color:white;display:grid;place-content:center;text-align:center;z-index:5}
button{background:#42e783;padding:15px 25px;border:0;border-radius:9px;font-weight:bold;font-size:17px;cursor:pointer}
</style>
</head>
<body>
<div id="hud">
<h2>🏪 Mercado Simulator</h2>
<div id="money">Dinheiro: R$ 200,00</div>
<div id="cart">Carrinho: vazio</div>
<p>WASD — andar<br>Mouse — olhar<br>E — pegar produto<br>F — pagar</p>
</div>
<div id="msg">Bem-vindo ao supermercado!</div>
<div id="start">
<h1>🛒 Mercado Simulator 3D</h1>
<p>Faça suas compras neste supermercado!</p>
<button onclick="begin()">JOGAR</button>
</div>
<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0xaabfc5);
scene.fog=new THREE.Fog(0xaabfc5,25,55);

const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.1,100);
camera.position.set(0,1.7,12);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x777777,2));
const sun=new THREE.DirectionalLight(0xffffff,2.5);
sun.position.set(-5,12,8);
sun.castShadow=true;
scene.add(sun);

function bloco(w,h,d,color,x,y,z){
 const m=new THREE.Mesh(
  new THREE.BoxGeometry(w,h,d),
  new THREE.MeshStandardMaterial({color,roughness:.65})
 );
 m.position.set(x,y,z);
 m.castShadow=true;
 m.receiveShadow=true;
 scene.add(m);
 return m;
}

// Estrutura do mercado
bloco(32,.3,34,0xc6c6bd,0,-.2,0);
bloco(32,7,.3,0xe8e8e0,0,3.5,-17);
bloco(.3,7,34,0xe8e8e0,-16,3.5,0);
bloco(.3,7,34,0xe8e8e0,16,3.5,0);

// Iluminação
for(let x=-12;x<=12;x+=6){
 const luz=new THREE.PointLight(0xfff0d5,55,15);
 luz.position.set(x,5,0);
 scene.add(luz);
 bloco(3,.12,1,0xffffff,x,6.5,0);
}

// Prateleiras com vários produtos
const produtos=[];
const catalogo=[
 {nome:'Leite',preco:6,cor:0xf6f0db},
 {nome:'Cereal',preco:13,cor:0xf2b934},
 {nome:'Suco',preco:8,cor:0xdf382e},
 {nome:'Biscoito',preco:5,cor:0x3986d5},
 {nome:'Arroz',preco:9,cor:0xf5f3e8},
 {nome:'Chocolate',preco:6,cor:0x603921},
 {nome:'Refrigerante',preco:7,cor:0xc52d32},
 {nome:'Sabão',preco:10,cor:0x37a9d4}
];

for(let z of [-10,-3,4]){
 for(let x of [-7,0,7]){
  bloco(3,2.8,.7,0x687579,x,1.4,z);
  for(let y of [.5,1.3,2.1]){
   bloco(3.1,.1,.85,0xc7b59d,x,y,z+.15);
   for(let i=0;i<5;i++){
    const item=catalogo[(i+Math.round(x)+Math.round(z)+20+
      Math.round(y*10))%catalogo.length];
    const cor=catalogo[Math.abs((i+Math.round(x)+
      Math.round(z)+Math.round(y*10)))%catalogo.length].cor;
    const p=bloco(.36,.42,.35,cor,
      x-1.1+i*.55,y+.25,z+.25);
    p.userData={nome:item.nome,preco:item.preco,pego:false};
    produtos.push(p);
   }
  }
 }
}

// Caixa de pagamento
bloco(4,1.1,1.5,0x286b46,10,.55,-12);
bloco(3.8,.12,1.6,0xf2f2ed,10,1.15,-12);
bloco(1,.18,.8,0x252525,9.4,1.3,-12.2);
bloco(2,.6,.15,0x168449,10,2,-12.8);

// Carrinho visível à frente do jogador
const carrinho=new THREE.Group();
scene.add(carrinho);
const metal=new THREE.MeshStandardMaterial({color:0xaebbc0,metalness:.8,roughness:.3});
function barra(a,b,r=.025){
 const dir=new THREE.Vector3().subVectors(b,a);
 const m=new THREE.Mesh(new THREE.CylinderGeometry(r,r,dir.length(),8),metal);
 m.position.copy(a).add(b).multiplyScalar(.5);
 m.quaternion.setFromUnitVectors(new THREE.Vector3(0,1,0),dir.normalize());
 carrinho.add(m);
}
for(let x of [-.4,.4]){
 barra(new THREE.Vector3(x,.25,0),new THREE.Vector3(x,.8,-1));
 barra(new THREE.Vector3(x,.8,-1),new THREE.Vector3(x,1,-1.4));
}
for(let y of [.3,.45,.6,.75]){
 barra(new THREE.Vector3(-.4,y,-.1),new THREE.Vector3(.4,y,-.1),.015);
 barra(new THREE.Vector3(-.4,y,-1),new THREE.Vector3(.4,y,-1),.015);
 for(let x of [-.4,.4])
  barra(new THREE.Vector3(x,y,-.1),new THREE.Vector3(x,y,-1),.015);
}

let dinheiro=200;
let cesta=[];
let yaw=0,pitch=0;
let jogando=false,travado=false;
const teclas={};
const mensagem=document.getElementById('msg');

function begin(){
 jogando=true;
 document.getElementById('start').style.display='none';
 renderer.domElement.requestPointerLock();
}
document.addEventListener('pointerlockchange',()=>{
 travado=document.pointerLockElement===renderer.domElement;
});
renderer.domElement.addEventListener('click',()=>{
 if(jogando&&!travado)renderer.domElement.requestPointerLock();
});
document.addEventListener('mousemove',e=>{
 if(!travado)return;
 yaw-=e.movementX*.002;
 pitch-=e.movementY*.002;
 pitch=Math.max(-1.2,Math.min(1.2,pitch));
 camera.rotation.order='YXZ';
 camera.rotation.y=yaw;
 camera.rotation.x=pitch;
});
document.addEventListener('keydown',e=>{
 teclas[e.code]=true;
 if(e.code==='KeyE')pegar();
 if(e.code==='KeyF')pagar();
});
document.addEventListener('keyup',e=>teclas[e.code]=false);

function pegar(){
 const raio=new THREE.Raycaster();
 raio.setFromCamera(new THREE.Vector2(0,0),camera);
 const acertos=raio.intersectObjects(produtos);
 if(!acertos.length||acertos[0].distance>3){
  mensagem.textContent='Mire em um produto e chegue mais perto!';
  return;
 }
 const p=acertos[0].object;
 if(p.userData.pego)return;
 if(dinheiro<p.userData.preco){
  mensagem.textContent='Dinheiro insuficiente!';
  return;
 }
 p.userData.pego=true;
 cesta.push(p.userData);
 p.visible=false;
 mensagem.textContent='Você pegou: '+p.userData.nome;
 document.getElementById('cart').textContent=
  'Carrinho: '+cesta.length+' produtos';
}
function pagar(){
 if(camera.position.distanceTo(new THREE.Vector3(10,1.7,-10))>4){
  mensagem.textContent='Vá até o caixa para pagar!';
  return;
 }
 if(!cesta.length){
  mensagem.textContent='Seu carrinho está vazio!';
  return;
 }
 let total=cesta.reduce((s,p)=>s+p.preco,0);
 dinheiro-=total;
 document.getElementById('money').textContent=
  'Dinheiro: R$ '+dinheiro.toFixed(2).replace('.',',');
 mensagem.textContent='Compra paga! Total: R$ '+
  total.toFixed(2).replace('.',',');
 cesta=[];
 document.getElementById('cart').textContent='Carrinho: vazio';
}

function animar(){
 requestAnimationFrame(animar);
 if(jogando){
  let velocidade=.085;
  if(teclas.KeyW){camera.position.x-=Math.sin(yaw)*velocidade;camera.position.z-=Math.cos(yaw)*velocidade}
  if(teclas.KeyS){camera.position.x+=Math.sin(yaw)*velocidade;camera.position.z+=Math.cos(yaw)*velocidade}
  if(teclas.KeyA){camera.position.x-=Math.cos(yaw)*velocidade;camera.position.z+=Math.sin(yaw)*velocidade}
  if(teclas.KeyD){camera.position.x+=Math.cos(yaw)*velocidade;camera.position.z-=Math.sin(yaw)*velocidade}
  camera.position.x=THREE.MathUtils.clamp(camera.position.x,-14,14);
  camera.position.z=THREE.MathUtils.clamp(camera.position.z,-15,15);
  camera.position.y=1.7;
  const offset=new THREE.Vector3(0,-1.25,-1.2);
  offset.applyAxisAngle(new THREE.Vector3(0,1,0),yaw);
  carrinho.position.copy(camera.position).add(offset);
  carrinho.rotation.y=yaw;
 }
 renderer.render(scene,camera);
}
animar();
addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
