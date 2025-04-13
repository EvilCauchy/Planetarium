<script>
	import { XR, Controller, Hand, useHandJoint, pointerControls } from '@threlte/xr';
	import { T } from '@threlte/core';
	import { interactivity } from '@threlte/extras';
	import DynamicSphere from './DynamicSphere.svelte';
	import { Text3DGeometry } from '@threlte/extras'

	// Initialize XR input and interactivity.
	const joint = useHandJoint('left', 'wrist');
	pointerControls('right');
	interactivity();


	const radii = [0.4, 0.7, 1];
	let spheres = [];
	let sphereCounter = 0;

	const spawnSphere = (event) => {
		const point = event.detail?.point;
		const position = point
			? [point.x, point.y, point.z]
			: [
					Math.random() * 7 + 2,
					Math.random() * 5 + 2,
					Math.random() * 7 + 2
			  ];
		const currentRadius = radii[sphereCounter % radii.length];
		spheres = [
			...spheres,
			{
				id: sphereCounter,
				position,
				radius: currentRadius
			}
		];
		sphereCounter++;
		console.log('Spawned sphere:', { position, currentRadius });
	};



</script>


<T.PerspectiveCamera
	makeDefault
	position={[13, 13, 13]}
	oncreate={(ref) => ref.lookAt(0, 1, 0)}
/>
<T.DirectionalLight position={[0, 15, 15]} castShadow />
<T.AmbientLight />


<T.Mesh
	rotation.x={-Math.PI / 2}
	receiveShadow
	position={[0, 0, 0]}
	onclick={spawnSphere}
>
	<T.CircleGeometry args={[15, 10]} />
	<T.MeshStandardMaterial color="white" />
</T.Mesh>


{#each spheres as sphere (sphere.id)}
	<DynamicSphere position={sphere.position} radius={sphere.radius} />
{/each}



<!-- XR components -->
<XR />
<Controller left />
<Controller right />
<Hand left />
<Hand right />