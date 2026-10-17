
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Supermercado 3D</title>
<style>
body{margin:0;overflow:hidden;font-family:Arial}
#info{position:fixed;top:10px;left:10px;background:#10251c;color:white;padding:15px;border-radius:12px;z-index:2}
#info h2{margin:0 0 8px;color:#72ff9a}
#start{position:fixed;inset:0;background:#13271f;color:white;display:grid;place-content:center;text-align:center;z-index:5}
button{padding:15px;background:#45df82;border:0;border-radius:9px;font-size:18px;cursor:pointer}
</style>
</head>
<body>
<div id="info">
<h2>🛒 Supermercado 3D</h2>
<div id="money">Dinheiro: R$ 100,00</div>
<div id="msg">Pegue produtos com E</div>
<p>WASD: andar<br>Mouse: olhar<br>E: pegar produto</p>
</div>
<div id="start">
<h1>🏪 Supermercado 3D</h1>
<p>Explore o mercado e compre produtos!</p>
<button onclick="startGame()">ENTRAR NO MERCADO</button>
</div>
<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0xb8c8ce);

const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.1,100);
camera.position.set(0,1.7,8);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.shadowMap.enabled=true;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x666666,2));
const light=new THREE.DirectionalLight(0xffffff,3);
light.position.set(5,10,5);
light.castShadow=true;
scene.add(light);

function box(w,h,d,color,x,y,z){
 const obj=new THREE.Mesh(
  new THREE.BoxGeometry(w,h,d),
  new THREE.MeshStandardMaterial({color,roughness:.65})
 );
 obj.position.set(x,y,z);
 obj.castShadow=true;
 obj.receiveShadow=true;
 scene.add(obj);
 return obj;
}

// Piso e paredes
box(30,.2,30,0xbcbcb5,0,-.1,0);
box(30,5,.3,0xe7e7e0,0,2.5,-15);
box(.3,5,30,0xe7e7e0,-15,2.5,0);
box(.3,5,30,0xe7e7e0,15,2.5,0);

// Luzes do teto
for(let x=-10;x<=10;x+=5){
 const l=new THREE.PointLight(0xfff0d0,35,12);
 l.position.set(x,4,0);
 scene.add(l);
 box(3,.1,1,0xffffff,x,4.7,0);
}

// Prateleiras
for(let z of [-8,-2,4]){
 for(let x of [-6,0,6]){
  box(3,2.5,.7,0x777d80,x,1.25,z);
  for(let y of [.5,1.2,1.9])
   box(3.1,.08,.9,0xb9a48b,x,y,z+.1);
 }
}

// Produtos
const products=[];
const names=['Leite','Cereal','Suco','Biscoito','Arroz','Café',
             'Refrigerante','Chocolate','Maçã','Pão'];
const colors=[0xf4f0df,0xe6b93d,0xe83b32,0x3979d2,
              0xf5f5f5,0x593b29,0x42a6d8,0x713e26,
              0xe7352b,0xc98d43];
const prices=[5,12,7,4,8,15,6,5,3,6];

for(let i=0;i<60;i++){
 const row=Math.floor(i/20);
 const col=i%20;
 const side=Math.floor(col/10);
 const x=(side===0?-6:6)-1.1+(col%10)*.24;
 const z=[-8,-2,4][row]+.55;
 const h=.35+(i%3)*.1;
 const p=box(.22,h,.25,colors[i%10],x,.35+(i%3)*.65,z);
 p.userData={
  name:names[i%10],
  price:prices[i%10],
  taken:false
 };
 products.push(p);
}

// Caixa
box(3,1,1.5,0x28734a,9,.5,-11);
box(1,.2,.7,0x333333,9,1.1,-11);
box(3,.15,.3,0xeeeeee,9,1.1,-11.8);

let money=100,total=0;
const keys={};
let yaw=0,pitch=0,playing=false,locked=false;

function startGame(){
 document.getElementById('start').style.display='none';
 playing=true;
 renderer.domElement.requestPointerLock();
}

document.addEventListener('pointerlockchange',()=>{
 locked=document.pointerLockElement===renderer.domElement;
});

document.addEventListener('mousemove',e=>{
 if(!locked)return;
 yaw-=e.movementX*.002;
 pitch-=e.movementY*.002;
 pitch=Math.max(-1.2,Math.min(1.2,pitch));
 camera.rotation.order='YXZ';
 camera.rotation.y=yaw;
 camera.rotation.x=pitch;
});

document.addEventListener('keydown',e=>{
 keys[e.code]=true;
 if(e.code==='KeyE')takeProduct();
});
document.addEventListener('keyup',e=>keys[e.code]=false);

function takeProduct(){
 const ray=new THREE.Raycaster();
 ray.setFromCamera(new THREE.Vector2(0,0),camera);
 const hits=ray.intersectObjects(products);
 const msg=document.getElementById('msg');

 if(!hits.length||hits[0].distance>3){
  msg.textContent='Chegue perto e mire em um produto!';
  return;
 }

 const p=hits[0].object;
 if(p.userData.taken)return;

 const data=p.userData;
 if(money<data.price){
  msg.textContent='Dinheiro insuficiente!';
  return;
 }

 data.taken=true;
 money-=data.price;
 total+=data.price;
 p.visible=false;

 document.getElementById('money').textContent=
  'Dinheiro: R$ '+money.toFixed(2).replace('.',',');

 msg.textContent='Comprou '+data.name+
  ' por R$ '+data.price.toFixed(2).replace('.',',');
}

function animate(){
 requestAnimationFrame(animate);

 if(playing){
  const speed=.09;
  if(keys.KeyW) {
   camera.position.x-=Math.sin(yaw)*speed;
   camera.position.z-=Math.cos(yaw)*speed;
  }
  if(keys.KeyS) {
   camera.position.x+=Math.sin(yaw)*speed;
   camera.position.z+=Math.cos(yaw)*speed;
  }
  if(keys.KeyA) {
   camera.position.x-=Math.cos(yaw)*speed;
   camera.position.z+=Math.sin(yaw)*speed;
  }
  if(keys.KeyD) {
   camera.position.x+=Math.cos(yaw)*speed;
   camera.position.z-=Math.sin(yaw)*speed;
  }
  camera.position.x=Math.max(-14,Math.min(14,camera.position.x));
  camera.position.z=Math.max(-14,Math.min(14,camera.position.z));
  camera.position.y=1.7;
 }
 renderer.render(scene,camera);
}

animate();

addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>

