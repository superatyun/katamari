<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Mini Katamari</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div id="ui">Size: <span id="size">1</span></div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script src="main.js"></script>
</body>
</html>
body {
  margin: 0;
  overflow: hidden;
  background: #87ceeb;
  font-family: sans-serif;
}

#ui {
  position: absolute;
  top: 10px;
  left: 10px;
  background: white;
  padding: 10px;
  border-radius: 10px;
}
// シーン作成
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 地面
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(200, 200),
  new THREE.MeshBasicMaterial({ color: 0x228B22 })
);
ground.rotation.x = -Math.PI / 2;
scene.add(ground);

// プレイヤー（塊）
let size = 1;
const katamariGeometry = new THREE.SphereGeometry(size, 32, 32);
const katamariMaterial = new THREE.MeshBasicMaterial({ color: 0xff00ff });
const katamari = new THREE.Mesh(katamariGeometry, katamariMaterial);
katamari.position.y = size;
scene.add(katamari);

// カメラ位置
camera.position.set(0, 5, 10);

// 小物生成
const objects = [];
for (let i = 0; i < 50; i++) {
  const objSize = Math.random() * 0.5 + 0.2;
  const obj = new THREE.Mesh(
    new THREE.BoxGeometry(objSize, objSize, objSize),
    new THREE.MeshBasicMaterial({ color: Math.random() * 0xffffff })
  );
  obj.position.set(
    (Math.random() - 0.5) * 100,
    objSize / 2,
    (Math.random() - 0.5) * 100
  );
  obj.userData.size = objSize;
  scene.add(obj);
  objects.push(obj);
}

// 操作
const keys = {};
window.addEventListener("keydown", e => keys[e.key] = true);
window.addEventListener("keyup", e => keys[e.key] = false);

// アニメーション
function animate() {
  requestAnimationFrame(animate);

  const speed = 0.1;
  if (keys["w"]) katamari.position.z -= speed;
  if (keys["s"]) katamari.position.z += speed;
  if (keys["a"]) katamari.position.x -= speed;
  if (keys["d"]) katamari.position.x += speed;

  // 衝突チェック
  objects.forEach((obj, index) => {
    if (!obj.visible) return;
    const distance = katamari.position.distanceTo(obj.position);
    if (distance < size && obj.userData.size < size) {
      obj.visible = false;
      size += obj.userData.size * 0.2;
      katamari.scale.set(size, size, size);
      document.getElementById("size").textContent = size.toFixed(2);
    }
  });

  // カメラ追従
  camera.position.x = katamari.position.x;
  camera.position.z = katamari.position.z + 10;
  camera.lookAt(katamari.position);

  renderer.render(scene, camera);
}

animate();
