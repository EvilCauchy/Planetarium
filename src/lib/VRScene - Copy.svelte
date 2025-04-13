<script>
	import { T, useTask } from '@threlte/core';
	import { Spring } from 'svelte/motion';
  

	export let position = [0, 1, 0];
	export let radius = 1;
	let currentPosition = [...position];
  

	const scale = new Spring(1);
  

	const initialX = position[0],
		  initialZ = position[2];
	const orbitRadius = Math.sqrt(initialX ** 2 + initialZ ** 2);
	const initialAngle = Math.atan2(initialZ, initialX);
	let orbitRotation = 0;
	const orbitSpeed = 0.1;
  

	let isDragging = false;
	function pointerDownHandler(event) {
	  isDragging = true;
	  event.stopPropagation();
	}
	function pointerMoveHandler(event) {
	  if (isDragging && event.detail?.point) {
		const newPoint = event.detail.point;
		currentPosition = [newPoint.x, currentPosition[1], newPoint.z];
	  }
	}
	function pointerUpHandler(event) {
	  isDragging = false;
	}
  

	function pointerEnterHandler(event) {
	  console.log("pointer entered sphere", event);
	  scale.target = 1.2;
	  event.stopPropagation();
	}
	function pointerLeaveHandler(event) {
	  console.log("pointer left sphere", event);
	  scale.target = 1;
	  event.stopPropagation();
	}
  
	// ===============================================
	// Update orbit position (if not dragging)
	// ===============================================
	useTask((delta) => {
	  if (!isDragging) {
		orbitRotation += delta * orbitSpeed;
		currentPosition = [
		  orbitRadius * Math.cos(initialAngle + orbitRotation),
		  currentPosition[1],
		  orbitRadius * Math.sin(initialAngle + orbitRotation)
		];
	  }
	});


	let t = 0;
	const colorSpeed = 0.5;
	useTask((delta) => {
	  t += delta * colorSpeed;
	});
	const colorStops = [
	  [0, 0, 255],   // Blue
	  [255, 0, 0],   // Red
	  [0, 255, 0],   // Green
	  [255, 255, 0]  // Yellow
	];
	$: cycle = t % colorStops.length;
	$: currentIndex = Math.floor(cycle);
	$: nextIndex = (currentIndex + 1) % colorStops.length;
	$: factor = cycle - currentIndex;
	function interpolateColor(factor, color1, color2) {
	  const r = Math.round((1 - factor) * color1[0] + factor * color2[0]);
	  const g = Math.round((1 - factor) * color1[1] + factor * color2[1]);
	  const b = Math.round((1 - factor) * color1[2] + factor * color2[2]);
	  return '#' + [r, g, b].map(x => x.toString(16).padStart(2, '0')).join('');
	}
	$: currentColor = interpolateColor(factor, colorStops[currentIndex], colorStops[nextIndex]);
  </script>
  
  <!--
	Bind the mesh’s scale to the Spring’s current value.
	The Spring updates smoothly over time when its .target changes.
  -->
  <T.Mesh 
	position={currentPosition}
	scale={scale.current}
	on:pointerdown={pointerDownHandler}
	on:pointermove={pointerMoveHandler}
	on:pointerup={pointerUpHandler}
	on:pointerover={pointerEnterHandler}   <!-- or pointerenter if preferred -->
	on:pointerout={pointerLeaveHandler}     <!-- or pointerleave -->
	castShadow>
	<T.SphereGeometry args={[radius, 32, 32]} />
	<T.MeshStandardMaterial color={currentColor} />
  </T.Mesh>
  