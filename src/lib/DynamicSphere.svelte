<script>
	import { T, useTask } from '@threlte/core';
	import { Spring } from 'svelte/motion';

	// Props: initial position and sphere radius.
	export let position = [0, 0, 0];
	export let radius = 1;

	// ---------------------------
	// HOVER & SCALING LOGIC
	// ---------------------------
	// Create a spring for smooth scaling (default value 1).
	const scaleSpring = new Spring(1);

	// Use a local flag to indicate whether the sphere is hovered.
	let hovered = false;

	// ---------------------------
	// ORBITING LOGIC (around [0, 0, 0])
	// ---------------------------
	// Use the provided position as the initial offset from the origin.
	const initialOffset = position;

	// Helper functions for vector math.
	function length(v) {
		return Math.sqrt(v[0] ** 2 + v[1] ** 2 + v[2] ** 2);
	}
	function cross(a, b) {
		return [
			a[1] * b[2] - a[2] * b[1],
			a[2] * b[0] - a[0] * b[2],
			a[0] * b[1] - a[1] * b[0]
		];
	}
	function normalize(v) {
		const l = length(v);
		return l ? [v[0] / l, v[1] / l, v[2] / l] : [0, 0, 0];
	}

	// Calculate the orbit radius as the length of the initial offset.
	const orbitRadius = length(initialOffset);
	const up = [0, 1, 0];
	let orbitAxis = cross(initialOffset, up);
	if (length(orbitAxis) < 0.001) {
		orbitAxis = cross(initialOffset, [1, 0, 0]);
	}
	orbitAxis = normalize(orbitAxis);

	// Use a base speed so that orbit speed is inversely proportional to orbit radius.
	const baseSpeed = 2.0;
	const orbitSpeed = orbitRadius > 0 ? baseSpeed / orbitRadius : 0;

	// Accumulate rotation in radians. This only updates when the sphere is not hovered.
	let orbitRotation = 0;
	useTask((delta) => {
		if (!hovered) {
			orbitRotation += delta * orbitSpeed;
		}
	});

	// Rotate a vector v around a given axis by angle using Rodrigues’ rotation formula.
	function rotateVector(v, axis, angle) {
		const cosA = Math.cos(angle);
		const sinA = Math.sin(angle);
		const dot = v[0] * axis[0] + v[1] * axis[1] + v[2] * axis[2];
		return [
			v[0] * cosA + (axis[1] * v[2] - axis[2] * v[1]) * sinA + axis[0] * dot * (1 - cosA),
			v[1] * cosA + (axis[2] * v[0] - axis[0] * v[2]) * sinA + axis[1] * dot * (1 - cosA),
			v[2] * cosA + (axis[0] * v[1] - axis[1] * v[0]) * sinA + axis[2] * dot * (1 - cosA)
		];
	}
	$: currentPosition = orbitRadius > 0
		? rotateVector(initialOffset, orbitAxis, orbitRotation)
		: initialOffset;

	// ---------------------------
	// COLOR INTERPOLATION LOGIC
	// ---------------------------
	let t = 0;
	const colorSpeed = 0.5;
	// Only update the color timer when not hovered.
	useTask((delta) => {
		if (!hovered) {
			t += delta * colorSpeed;
		}
	});
	// Define a set of color stops as RGB arrays.
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
		return '#' + [r, g, b].map((x) => x.toString(16).padStart(2, '0')).join('');
	}
	$: currentColor = interpolateColor(factor, colorStops[currentIndex], colorStops[nextIndex]);
</script>

<T.Mesh
	position={currentPosition}
	castShadow
	scale={scaleSpring.current}
	onpointerenter={() => {
		scaleSpring.target = 1.3;
		hovered = true;
	}}
	onpointerleave={() => {
		scaleSpring.target = 1;
		hovered = false;
	}}
>
	<T.SphereGeometry args={[radius, 32, 32]} />
	<T.MeshStandardMaterial color={currentColor} />
</T.Mesh>

