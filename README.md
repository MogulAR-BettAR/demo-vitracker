# vitracker v1.0.2

**vitracker** is a Image Targets feature of **bettar-vidometry** library.

This library is created to simplify using AR image targets experience on the web.
In a few lines of the code, you can integrate the Image Targets experience into your web application to provide an exciting AR experience for the users.

## vitracker-demo

This example project shows how to integrate a **vitracker** into your web application.

You can check [live example](https://bettar.life/vitracker/).

### Installation

Execute the following command in order to install dependencies:

```tsx
npm i
```

or

```tsx
yarn install
```

### Development

Execute the following command in order to run the application in development mode:

start - runs example with static **vidometry-vitracker** tag;

```tsx
npm run start
```

OR

```tsx
yarn start
```

Run the following URL on your mobile device:

```tsx
https://localhost:8080
```

**As an image target for this demo you can use the following image:**

![https://bettar.life/mona/mona.jpg](https://bettar.life/mona/mona.jpg)

# vitracker integration

In order to add a **vitracker** to your site you need the following actions:

1. Add the following JS script in the **head** section:

```tsx
<head>
	...
	<script src="https://bettar.life/vidometry/vitracker.1.0.2.js"></script>
	...
</head>
```

1. Add **vitracker** to web page:
    
    a. add **viomdetry-vitracker** tag:
    

```tsx
<vidometry-vitracker id="vitracker"></vidometry-vitracker>
```

b. add **vidometry-vitracker** programmatically:

```tsx
vitracker = this.document.createElement('vidometry-vitracker');
document.body.appendChild(vitracker);
```

 

1. Add **vitracker** callbacks:
    1. **onProcess(rototranslation, focal, targetId)** - uses to update 3d position of the perspective camera of the scene. Throws every frame;
        1. **rototranslation** - source of rototranslation matrix (4x4);
        2. **focal** - focal length of the perspective camera;
        3. **targetId** - id of the image target (-1 if image target isn’t detected);

```jsx
vitracker.onProcess = onVitrackerProcess;
```

     Image target is detected then targetId isn’t equal to -1. If the image target is not detected then targetId will be -1.

1. Initialize **vitracker**:
    1. **sceneWidth** - width of the scene in pixels;
    2. **sceneHeight** - height of the scene in pixels;
    3. **videoCanvas** - reference to the canvas tag where video frame should be rendered;
    4. returns **Promise;**

```jsx
vitracker.initialize(sceneWidth, sceneHeight, videoCanvas);
```

1. Load target:
    
    As soon as **vitracker** is initialized you have to load an image target which will be tracked.
    
    1. **src** - the source of the image target (*.jpg);
    2. returns **Promise<Target>**
        1. **Target** - object {id, width, height}:
            1. id - id of the image target;
            2. width - width of the image target;
            3. height - height of the image target;

```tsx
vitracker.addTarget(src);
```

1. Train **vitracker**:
    
    When the target is added, you must train the vitracker.
    
    1. returns **Promise**;

```jsx
vitracker.train();
```

After the training experience is ready to work.

**Complete example with vidometry-vitracker tag:** 

```jsx
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Vitracker Demo</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.js"
    integrity="sha512-NLtnLBS9Q2w7GKK9rKxdtgL7rA7CAS85uC/0xd9im4J/yOL4F9ZVlv634NAM7run8hz3wI2GabaA6vv8vJtHiQ=="
    crossorigin="anonymous" referrerpolicy="no-referrer"></script>
  <script src="https://bettar.life/vidometry/vitracker.1.0.2.js"></script>
  <script>
    class Scene3D {
      constructor(width, height, fov, canvas) {
        this.fov = fov;
        this.width = width;
        this.height = height;

        // 1. renderer
        this.renderer = new THREE.WebGLRenderer({
          canvas,
          antialias: true,
          alpha: true,
        });
        this.renderer.setSize(width, height);

        // 2. scene
        this.scene = new THREE.Scene();

        // 3. camera
        const aspect = width / height;
        this.camera = new THREE.PerspectiveCamera(fov, aspect, 0.01, 10000);
        this.camera.filmGauge = Math.max(width, height);
        this.camera.updateProjectionMatrix();
        this.scene.add(this.camera);

        // 4. container
        this.container = new THREE.Object3D();
        this.container.frustumCulled = false;
        this.container.matrixAutoUpdate = false;
        this.scene.add(this.container);
      }

      updateFocal(focal) {
        if (this.focal !== focal) {
          this.focal = focal;
          this.camera.setFocalLength(focal);
          this.camera.updateProjectionMatrix();
        }
      }

      show() {
        this.container.visible = true;
      }

      hide() {
        this.container.visible = false;
      }

      addLight(light) {
        this.scene.add(light);
      }

      add(object) {
        this.container.add(object);
      }

      updateObjectMatrix(roto) {
        const matrix = new THREE.Matrix4();
        matrix.set(
          roto[0], roto[1], roto[2], roto[3],
          roto[4], roto[5], roto[6], roto[7],
          roto[8], roto[9], roto[10], roto[11],
          roto[12], roto[13], roto[14], roto[15],
        );

        matrix.decompose(this.container.position, this.container.quaternion, this.container.scale);
        this.container.updateMatrix();
      }

      render(roto) {
        const matrix = new THREE.Matrix4();
        matrix.set(
          roto[0], roto[1], roto[2], roto[3],
          roto[4], roto[5], roto[6], roto[7],
          roto[8], roto[9], roto[10], roto[11],
          roto[12], roto[13], roto[14], roto[15],
        );

        matrix.decompose(this.camera.position, this.camera.quaternion, this.camera.scale);
        this.renderer.render(this.scene, this.camera);
      }

    };

    var scene, target, width, height, fov;
    var isReady = false;

    const onVitrackerProcess = (roto, focal, tragetId) => {
      if (tragetId === -1) {
        scene.hide();
      } else {
        scene.show();
      }

      scene.updateFocal(focal);
      scene.render(roto);
    }

    function init() {
      width = window.innerWidth;
      height = window.innerHeight;
      fov = 65;

      initVitracker();
      initScene();
    }

    function initVitracker() {
      const canvas = document.getElementById('canvas-video');
      const vitracker = document.getElementById('vitracker');
      vitracker.onProcess = onVitrackerProcess;
      vitracker.initialize(width, height, canvas)
        .then(() => {
          return vitracker.addTarget('./assets/mona.jpg');
        })
        .then((t) => {
          target = t;
          console.log('target loaded; w: ' + target.width + '; h: ' + target.height);
          addOverlay();
          return vitracker.train();
        })
        .then(() => {
          console.log('trained!');
          isReady = true;
        });
    }

    function initScene() {
      const canvas = document.getElementById('canvas-gl');
      canvas.width = width;
      canvas.height = height;
      scene = new Scene3D(width, height, fov, canvas);
    }

    function addOverlay() {
      const height = 1;
      const width = height * target.width / target.height;
      const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });
      material.transparent = true;
      material.opacity = 0.5;
      const plane = new THREE.Mesh(new THREE.BoxGeometry(width, height, 0.001,), material);
      plane.position.set(0, height / 2, 0);
      scene.add(plane);
    }
  </script>
</head>

<body onload="init();" onclick="onClick();">
  <div style="position: fixed; left: 0; top: 0; width: 100vw; height: 100vh;" scrolling="no">
    <vidometry-vitracker id="vitracker"></vidometry-vitracker>
    <canvas id="canvas-video" style="position: absolute;"></canvas>
    <canvas id="canvas-gl" style="position: absolute; left: 0; top: 0; width: 100vw; height: 100vh;" width="100vw"
      height="100vh"></canvas>
  </div>
</body>

</html>
```

## API

### Methods:

**initialize(sceneWidth, sceneHeight, videoCanvas)** - initializes **vitracker**;

1. ***sceneWidth*** - width of the scene in pixels;
2. ***sceneHeight*** - height of the scene in pixels;
3. ***videoCanvas*** - reference to the canvas tag where video frame should be rendered;

**addTarget(src)** - loads image target:

1. **src** - the source of the image target (*.jpg);
2. returns **Promise<Target>**
    1. **Target** - object {id, width, height}:
        1. id - id of the image target;
        2. width - width of the image target;
        3. height - height of the image target;

**train()** - trains **vitracker**;

1. returns **Promise**;

### Callbacks

**onProcess(rototranslation, focal, targetId)** - uses to update the 3d position of the perspective camera of the scene. Throws every frame;

1. **rototranslation** - the source of rototranslation matrix (4x4);
2. **focal** - focal length of the perspective camera;
3. **targetId** - id of the image target (-1 if image target isn’t detected).