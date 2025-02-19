---
id: threejs-face
title: ThreeJS Face Tracking
sidebar_label: ThreeJS Face
---

import {customFields} from '/docusaurus.config.js';
import useBaseUrl from '@docusaurus/useBaseUrl';

This is sample application using three.js instead of AFRAME. The effect should be similar to <a href="../face-tracking-examples/minimal">Minimal</a>

## Try it out
<a href={useBaseUrl('/face-tracking-samples/three.html')} target="_blank">Live Demo</a>

<code>
{`
<html>
  <head>
    <script async src="https://unpkg.com/es-module-shims@1.3.6/dist/es-module-shims.js"></script>
    <script type="importmap">
    {
      "imports": {
	"three": "https://unpkg.com/three@0.147.0/build/three.module.js",
	"three/addons/": "https://unpkg.com/three@0.147.0/examples/jsm/",
	"mindar-face-three":"https://cdn.jsdelivr.net/npm/mind-ar@1.2.0/dist/mindar-face-three.prod.js"
      }
    }
    </script>
    <script type="module">
      import * as THREE from 'three';
      import { MindARThree } from 'mindar-face-three';
      const mindarThree = new MindARThree({
	container: document.querySelector("#container"),
      });
      const {renderer, scene, camera} = mindarThree;
      const anchor = mindarThree.addAnchor(1);
      const geometry = new THREE.SphereGeometry( 0.1, 32, 16 );
      const material = new THREE.MeshBasicMaterial( {color: 0x00ffff, transparent: true, opacity: 0.5} );
      const sphere = new THREE.Mesh( geometry, material );
      anchor.group.add(sphere);
      const start = async() => {
	await mindarThree.start();
	renderer.setAnimationLoop(() => {
	  renderer.render(scene, camera);
	});
      }
      const startButton = document.querySelector("#startButton");
      startButton.addEventListener("click", () => {
	start();
      });
      stopButton.addEventListener("click", () => {
	mindarThree.stop();
	mindarThree.renderer.setAnimationLoop(null);
      });
    </script>
    <style>
      body {
	margin: 0;
      }
      #container {
	width: 100vw;
	height: 100vh;
	position: relative;
	overflow: hidden;
      }
      #control {
	position: fixed;
	top: 0;
	left: 0;
	z-index: 2;
      }
    </style>
  </head>
  <body>
    <div id="control">
      <button id="startButton">Start</button>
      <button id="stopButton">Stop</button>
    </div>
    <div id="container">
    </div>
  </body>
</html>
`}
</code>
