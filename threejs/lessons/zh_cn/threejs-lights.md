Three.js灯
本文是有关three.js的一系列文章的一部分。第一篇文章是three.js基础知识。如果您还没有读过，并且还不熟悉Three.js，则可能要考虑从这里开始，以及有关设置环境的文章。在 前面的文章是关于纹理。

让我们讨论一下如何使用三种光源。

从我们之前的示例之一开始，让我们更新相机。我们将视场设置为45度，将远端平面设置为100个单位，并将摄像机从原点向上移动10个单位，向后移动20个单位

const fov = 45;
const aspect = 2;  //画布默认
const near = 0.1;
const far = 100;
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
camera.position.set(0, 10, 20);
接下来让我们添加OrbitControls。OrbitControls让用户绕镜头旋转或绕某点旋转。这OrbitControls是three.js的可选功能，因此首先我们需要将它们包括在我们的页面中

import * as THREE from './resources/three/r122/build/three.module.js';
import {OrbitControls} from './resources/threejs/r122/examples/jsm/controls/OrbitControls.js';
然后我们可以使用它们。我们通过OrbitControls一个摄像机来控制，使用DOM元素来获取输入事件

const controls = new OrbitControls(camera, canvas);
controls.target.set(0, 5, 0);
controls.update();
我们还将目标设置为绕原点高出5个单位绕行，然后调用controls.update以使控件使用新目标。

接下来，让我们点亮一些内容。首先，我们将制作地平面。我们将应用一个很小的2x2像素棋盘格纹理，如下所示


首先，我们加载纹理，将其设置为重复，将过滤设置为最接近，并设置我们希望其重复多少次。由于纹理是2x2像素的棋盘格，因此通过重复并将重复设置为平面大小的一半，棋盘格上的每个检查将恰好是1个单位大；

const planeSize = 40;
 
const loader = new THREE.TextureLoader();
const texture = loader.load('resources/images/checker.png');
texture.wrapS = THREE.RepeatWrapping;
texture.wrapT = THREE.RepeatWrapping;
texture.magFilter = THREE.NearestFilter;
const repeats = planeSize / 2;
texture.repeat.set(repeats, repeats);
然后，我们创建一个平面几何图形，该平面的材质以及一个将其插入场景的网格。平面默认为在XY平面中，但地面在XZ平面中，因此我们将其旋转。

const planeGeo = new THREE.PlaneBufferGeometry(planeSize, planeSize);
const planeMat = new THREE.MeshPhongMaterial({
  map: texture,
  side: THREE.DoubleSide,
});
const mesh = new THREE.Mesh(planeGeo, planeMat);
mesh.rotation.x = Math.PI * -.5;
scene.add(mesh);
让我们添加一个立方体和一个球体，这样我们要照亮3件事，包括平面

{
  const cubeSize = 4;
  const cubeGeo = new THREE.BoxBufferGeometry(cubeSize, cubeSize, cubeSize);
  const cubeMat = new THREE.MeshPhongMaterial({color: '#8AC'});
  const mesh = new THREE.Mesh(cubeGeo, cubeMat);
  mesh.position.set(cubeSize + 1, cubeSize / 2, 0);
  scene.add(mesh);
}
{
  const sphereRadius = 3;
  const sphereWidthDivisions = 32;
  const sphereHeightDivisions = 16;
  const sphereGeo = new THREE.SphereBufferGeometry(sphereRadius, sphereWidthDivisions, sphereHeightDivisions);
  const sphereMat = new THREE.MeshPhongMaterial({color: '#CA8'});
  const mesh = new THREE.Mesh(sphereGeo, sphereMat);
  mesh.position.set(-sphereRadius - 1, sphereRadius + 2, 0);
  scene.add(mesh);
}
现在我们有一个要点亮的场景，让我们添加灯光！

AmbientLight
首先，让我们 AmbientLight

const color = 0xFFFFFF;
const intensity = 1;
const light = new THREE.AmbientLight(color, intensity);
scene.add(light);
我们还要进行调整，以便调整灯光的参数。我们将再次使用dat.GUI。为了能够通过dat.GUI调整颜色，我们需要一个小的助手来为dat.GUI呈现一个属性，该属性看起来像CSS十六进制颜色字符串（例如：）#FF8844。我们的助手将从指定属性中获取颜色，将其转换为十六进制字符串以提供给dat.GUI。当dat.GUI尝试设置助手的属性时，我们会将结果分配回灯光的颜色。

这是帮手：

class ColorGUIHelper {
  constructor(object, prop) {
    this.object = object;
    this.prop = prop;
  }
  get value() {
    return `#${this.object[this.prop].getHexString()}`;
  }
  set value(hexString) {
    this.object[this.prop].set(hexString);
  }
}
这是我们设置dat.GUI的代码

const gui = new GUI();
gui.addColor(new ColorGUIHelper(light, 'color'), 'value').name('color');
gui.add(light, 'intensity', 0, 2, 0.01);
这就是结果


click here to open in a separate window
在场景中单击并拖动以使摄像机旋转。

注意没有定义。形状是平坦的。在AmbientLight有效刚刚乘以材料的颜色和灯光的颜色倍的强度。

color = materialColor * light.color * light.intensity;
而已。它没有方向。实际上，这种类型的环境照明并不能像照明一样有用，因为它是100％的照明，除了改变场景中所有物体的颜色外，它看起来并不像照明。它确实可以帮助使暗处不要太暗。

HemisphereLight
让我们将代码切换为HemisphereLight。AHemisphereLight 可以采用天空色和底色，并将材质的颜色乘以这两种颜色之间的乘积-如果对象的表面朝上，则为天空色；如果对象的表面朝下，则为底色。

这是新的代码

const color = 0xFFFFFF;
const skyColor = 0xB1E1FF;  // 浅蓝
const groundColor = 0xB97A20;  //棕橙色
const intensity = 1;
const light = new THREE.AmbientLight(color, intensity);
const light = new THREE.HemisphereLight(skyColor, groundColor, intensity);
scene.add(light);
我们还要更新dat.GUI代码以编辑两种颜色

const gui = new GUI();
gui.addColor(new ColorGUIHelper(light, 'color'), 'value').name('color');
gui.addColor(new ColorGUIHelper(light, 'color'), 'value').name('skyColor');
gui.addColor(new ColorGUIHelper(light, 'groundColor'), 'value').name('groundColor');
gui.add(light, 'intensity', 0, 2, 0.01);
结果：


click here to open in a separate window
再次注意，几乎没有定义，一切看起来都比较平坦。该HemisphereLight组合使用另一种光可以帮助提供一个很好的那种天空和地面的颜色的影响。这样，最好将其与其他光源或替代物结合使用AmbientLight。

DirectionalLight
让我们将代码切换为DirectionalLight。ADirectionalLight通常用于表示太阳。

const color = 0xFFFFFF;
const intensity = 1;
const light = new THREE.DirectionalLight(color, intensity);
light.position.set(0, 10, 0);
light.target.position.set(-5, 0, 0);
scene.add(light);
scene.add(light.target);
请注意，我们必须将light和添加light.target 到场景中。three.jsDirectionalLight将朝着目标方向发光。

让我们来做，以便可以通过将目标添加到我们的GUI中来移动它。

const gui = new GUI();
gui.addColor(new ColorGUIHelper(light, 'color'), 'value').name('color');
gui.add(light, 'intensity', 0, 2, 0.01);
gui.add(light.target.position, 'x', -10, 10);
gui.add(light.target.position, 'z', -10, 10);
gui.add(light.target.position, 'y', 0, 10);

click here to open in a separate window
很难知道发生了什么。Three.js有很多帮助对象，我们可以将它们添加到场景中以帮助可视化场景的不可见部分。在这种情况下，我们将使用 DirectionalLightHelper绘制一个平面来表示光，以及从光到目标的直线。我们只是将其传递给光源，然后将其添加到场景中。

const helper = new THREE.DirectionalLightHelper(light);
scene.add(helper);
当我们在做它时，我们可以设置光源和目标的位置。要做到这一点，我们将做一个函数，给定Vector3会调整其x，y和z使用性能dat.GUI。

function makeXYZGUI(gui, vector3, name, onChangeFn) {
  const folder = gui.addFolder(name);
  folder.add(vector3, 'x', -10, 10).onChange(onChangeFn);
  folder.add(vector3, 'y', 0, 10).onChange(onChangeFn);
  folder.add(vector3, 'z', -10, 10).onChange(onChangeFn);
  folder.open();
}
请注意，只要update我们更改某些内容，就需要调用帮助程序的函数，以便帮助程序知道自己进行更新。因此，我们传入了一个onChangeFn随时可以调用的函数dat.GUI更新了一个值。

然后我们可以像这样将它用于光源的位置和目标的位置

function updateLight() {
  light.target.updateMatrixWorld();
  helper.update();
}
updateLight();
 
const gui = new GUI();
gui.addColor(new ColorGUIHelper(light, 'color'), 'value').name('color');
gui.add(light, 'intensity', 0, 2, 0.01);
 
makeXYZGUI(gui, light.position, 'position', updateLight);
makeXYZGUI(gui, light.target.position, 'target', updateLight);
现在我们可以移动灯光及其目标


click here to open in a separate window
绕着相机旋转，看起来更容易。该平面表示a，DirectionalLight因为定向光会计算沿一个方向入射的光。光线毫无 意义，它是无限的光平面，可以射出平行的光线。

PointLight
APointLight是位于某个点的光线，并从该点向各个方向发射光线。让我们更改代码。

const color = 0xFFFFFF;
const intensity = 1;
const light = new THREE.DirectionalLight(color, intensity);
const light = new THREE.PointLight(color, intensity);
light.position.set(0, 10, 0);
light.target.position.set(-5, 0, 0);
scene.add(light);
scene.add(light.target);
让我们也切换到 PointLightHelper

const helper = new THREE.DirectionalLightHelper(light);
const helper = new THREE.PointLightHelper(light);
scene.add(helper);
由于没有目标，因此onChange功能可以更简单。

function updateLight() {
  light.target.updateMatrixWorld();
  helper.update();
}
updateLight();
请注意，在某种程度上，aPointLightHelper没有点。它只是画了一个小的线框钻石。它可以很容易地变成您想要的任何形状，只需在灯光本身上添加一个网格即可。

APointLight具有的附加属性distance。如果adistance为0，则PointLight照到无穷大。如果distance大于0，则光源在光源处发出其全部强度，而在distance 远离光源的单位处逐渐消失而无影响。

让我们设置GUI，以便我们可以调整距离。

const gui = new GUI();
gui.addColor(new ColorGUIHelper(light, 'color'), 'value').name('color');
gui.add(light, 'intensity', 0, 2, 0.01);
gui.add(light, 'distance', 0, 40).onChange(updateLight);
 
makeXYZGUI(gui, light.position, 'position', updateLight);
makeXYZGUI(gui, light.target.position, 'target', updateLight);
现在尝试一下。


click here to open in a separate window
请注意，当distance> 0时，光线将如何淡出。

SpotLight
聚光灯实际上是连接了圆锥体的点光源，该光源仅在圆锥体内部发光。实际上有2个锥体。外锥和内锥。在内锥和外锥之间，光从全强度衰减到零。

要使用，SpotLight我们需要一个像定向光一样的目标。灯光的圆锥体将朝向目标打开。

DirectionalLight从上方通过助手修改我们的

const color = 0xFFFFFF;
const intensity = 1;
const light = new THREE.DirectionalLight(color, intensity);
const light = new THREE.SpotLight(color, intensity);
scene.add(light);
scene.add(light.target);
 
const helper = new THREE.DirectionalLightHelper(light);
const helper = new THREE.SpotLightHelper(light);
scene.add(helper);
聚光灯的圆锥角angle 以弧度为单位设置。我们将使用DegRadHelper从 质地文章度呈现的UI。

gui.add(new DegRadHelper(light, 'angle'), 'value', 0, 90).name('angle').onChange(updateLight);
通过将penumbra属性设置为外锥的百分比来定义内锥。换句话说，当penumbra为0时，内部代码与外部锥体的大小相同（0 =无差异）。当a penumbra为1时，光线从圆锥的中心开始向外部圆锥逐渐消失。当penumbra为.5时，光线从外锥中心之间的50％开始衰减。

gui.add(light, 'penumbra', 0, 1, 0.01);

click here to open in a separate window
请注意，默认设置penumbra为0，聚光灯的边缘非常锋利，而当您将其调整为penumbra1时，边缘就会模糊。

可能很难看到聚光灯的圆锥形。原因是它在地下。将距离缩短到5左右，您会看到圆锥的开口端。

RectAreaLight
还有另一种类型的光，，RectAreaLight它确切地表示听起来像什么，矩形的光，如长荧光灯或天花板上的磨砂天光。

在RectAreaLight只能与MeshStandardMaterial和 MeshPhysicalMaterial让我们改变我们所有的材料MeshStandardMaterial

  ...
 
  const planeGeo = new THREE.PlaneBufferGeometry(planeSize, planeSize);
  const planeMat = new THREE.MeshPhongMaterial({
  const planeMat = new THREE.MeshStandardMaterial({
    map: texture,
    side: THREE.DoubleSide,
  });
  const mesh = new THREE.Mesh(planeGeo, planeMat);
  mesh.rotation.x = Math.PI * -.5;
  scene.add(mesh);
}
{
  const cubeSize = 4;
  const cubeGeo = new THREE.BoxBufferGeometry(cubeSize, cubeSize, cubeSize);
 const cubeMat = new THREE.MeshPhongMaterial({color: '#8AC'});
 const cubeMat = new THREE.MeshStandardMaterial({color: '#8AC'});
  const mesh = new THREE.Mesh(cubeGeo, cubeMat);
  mesh.position.set(cubeSize + 1, cubeSize / 2, 0);
  scene.add(mesh);
}
{
  const sphereRadius = 3;
  const sphereWidthDivisions = 32;
  const sphereHeightDivisions = 16;
  const sphereGeo = new THREE.SphereBufferGeometry(sphereRadius, sphereWidthDivisions, sphereHeightDivisions);
  const sphereMat = new THREE.MeshPhongMaterial({color: '#CA8'});
 const sphereMat = new THREE.MeshStandardMaterial({color: '#CA8'});
  const mesh = new THREE.Mesh(sphereGeo, sphereMat);
  mesh.position.set(-sphereRadius - 1, sphereRadius + 2, 0);
  scene.add(mesh);
}
要使用，RectAreaLight我们需要包括一些额外的three.js可选数据，并且将包括RectAreaLightHelper来帮助我们可视化灯光

import * as THREE from './resources/three/r122/build/three.module.js';
import {RectAreaLightUniformsLib} from './resources/threejs/r122/examples/jsm/lights/RectAreaLightUniformsLib.js';
import {RectAreaLightHelper} from './resources/threejs/r122/examples/jsm/helpers/RectAreaLightHelper.js';
我们需要打电话 RectAreaLightUniformsLib.init

function main() {
  const canvas = document.querySelector('#c');
  const renderer = new THREE.WebGLRenderer({canvas});
  RectAreaLightUniformsLib.init();
如果您忘记了数据，指示灯仍然可以工作，但是看起来很有趣，因此请务必记住包括额外的数据。

现在我们可以创造光

const color = 0xFFFFFF;
const intensity = 5;
const width = 12;
const height = 4;
const light = new THREE.RectAreaLight(color, intensity, width, height);
light.position.set(0, 10, 0);
light.rotation.x = THREE.MathUtils.degToRad(-90);
scene.add(light);
 
const helper = new RectAreaLightHelper(light);
light.add(helper);
要注意的一件事是，与DirectionalLight和和不同SpotLight，它们 RectAreaLight不使用目标。它只是使用其旋转。需要注意的另一件事是帮助者必须是光明的孩子。它不是像其他帮手那样的孩子。

让我们也调整GUI。我们将使它能够旋转灯并调整其width和height

const gui = new GUI();
gui.addColor(new ColorGUIHelper(light, 'color'), 'value').name('color');
gui.add(light, 'intensity', 0, 10, 0.01);
gui.add(light, 'width', 0, 20).onChange(updateLight);
gui.add(light, 'height', 0, 20).onChange(updateLight);
gui.add(new DegRadHelper(light.rotation, 'x'), 'value', -180, 180).name('x rotation').onChange(updateLight);
gui.add(new DegRadHelper(light.rotation, 'y'), 'value', -180, 180).name('y rotation').onChange(updateLight);
gui.add(new DegRadHelper(light.rotation, 'z'), 'value', -180, 180).name('z rotation').onChange(updateLight);
 
makeXYZGUI(gui, light.position, 'position', updateLight);
这就是。


click here to open in a separate window
我们没有介绍的一件事是WebGLRenderer 被叫上有一个设置physicallyCorrectLights。它影响光如何随着距光的距离而下降。它只会影响PointLight和SpotLight。RectAreaLight自动执行此操作。

对于灯，虽然基本思想是您没有设置淡出灯的距离，也没有设置intensity。取而代之的power是将光的流明设置为流明，然后three.js将使用物理计算，如真实的光。在这种情况下，three.js的单位是米，一个60w的灯泡大约有800流明。还有一个decay财产。应该将其设置2为现实衰减。

让我们测试一下。

首先，我们将打开物理上正确的灯光

const renderer = new THREE.WebGLRenderer({canvas});
renderer.physicallyCorrectLights = true;
然后我们将设置power为800流明，将其设置decay为2，将设置distance为Infinity。

const color = 0xFFFFFF;
const intensity = 1;
const light = new THREE.PointLight(color, intensity);
light.power = 800;
light.decay = 2;
light.distance = Infinity;
我们将添加gui，以便我们可以更改power和decay

const gui = new GUI();
gui.addColor(new ColorGUIHelper(light, 'color'), 'value').name('color');
gui.add(light, 'decay', 0, 4, 0.01);
gui.add(light, 'power', 0, 2000);

click here to open in a separate window
请务必注意，添加到场景中的每种光源都会减慢three.js渲染场景的速度，因此您应始终尝试使用尽可能少的光线来实现目标。

接下来，让我们继续处理相机。
