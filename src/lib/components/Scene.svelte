<script lang="ts">
  import { 
    T,
    useThrelte,
    useRender 
  } from '@threlte/core'
  import {
    OrbitControls,
    Instance,
    InstancedMesh
  } from '@threlte/extras';
  import {
    EffectComposer,
    EffectPass,
    RenderPass,
    BloomEffect,
    KernelSize
  } from 'postprocessing';
  
  const { scene, renderer, camera, size } = useThrelte()
  const composer = new EffectComposer(renderer)
  const setupEffectComposer = (camera) => {
    composer.removeAllPasses()
    composer.addPass(new RenderPass(scene, camera))
    composer.addPass(
      new EffectPass(
        camera,
        new BloomEffect({
          intensity: 2,
          luminanceThreshold: 0.08,
          height: 512,
          width: 512,
          luminanceSmoothing: 0.08,
          mipmapBlur: true,
          kernelSize: KernelSize.MEDIUM
        })
      )
    )
  }
  // We need to set up the passes according to the camera in use
  $: setupEffectComposer($camera)
  $: composer.setSize($size.width, $size.height)
  useRender((_, delta) => {
    composer.render(delta)
  })
  
  const count = 20000;
  const instances = generate(count);
  
  function generate(count) {
    const origin = vec3();
    const noise = vec3();
    const noiseFactor = 144;
    const thetaFactor = 1.00048;
    const points = new Array(count);
    
    let theta = 0.1, dist = 0.1;
    
    // Arm 1
    for (let i = 0; i < count / 2; i++) {
      randomise(noise, noiseFactor);
      const x = origin[0] + noise[0] + Math.cos(theta) * dist;
      const y = origin[1] + noise[1];
      const z = origin[2] + noise[2] + Math.sin(theta) * dist;
      theta *= thetaFactor;
      dist += 0.028 + (dist * 0.0002);
      points[i] = [
        ...vec3(x, y, z),
        ...getColor(dist)
       ];
    }
    
    dist = 0.1, theta = 0.1;
    // Arm 2 
    for (let i = count / 2; i < count; i++) {
      randomise(noise, noiseFactor);
      const x = origin[0] + noise[0] + -Math.cos(theta) * dist;
      const y = origin[1] + noise[1];
      const z = origin[2] + noise[2] + -Math.sin(theta) * dist;
      
      theta *= thetaFactor;
      dist += 0.028 + (dist * 0.0002);
      points[i] = [
        ...vec3(x, y, z),
        ...getColor(dist)
       ];
    }
    
    return points;
    
  }
  
  function vec3(x = 0, y = 0, z = 0) {
    return [ x, y, z ];
  }
  
  function scale(vector, scalar) {
    vector[0] *= scalar;
    vector[1] *= scalar;
    vector[2] *= scalar;
    return vector;
  }
  
  function randomise(vector, scalar) {
    vector[0] = Math.random();
    vector[1] = Math.random();
    vector[2] = Math.random();
    const dist = (-1 + Math.random() * 2) * scalar;
    return scale(vector, dist);
  }
  
  function getColor(dist) {
    const colorA = vec3(0.12, 0.13, 1.00);
    const colorB = vec3(1.00, 0.12, 0.28);
    const max = 900;
    const normalised = dist / max;
    return vec3(
      lerp(colorA[0], colorB[0], normalised),
      lerp(colorA[1], colorB[1], normalised),
      lerp(colorA[2], colorB[2], normalised)
    );
  }
  
  function lerp(a, b, t) {
    return a + (b - a) * t;
  }
  
</script>

<T.PerspectiveCamera
  makeDefault
  position={300}
  far={20000}
  on:create={({ ref }) => {
    ref.lookAt(0, 1, 0)
  }}
>
  <OrbitControls enableDamping />
</T.PerspectiveCamera>

<InstancedMesh limit={count} range={count}>
  <T.BoxGeometry args={[ 2, 2, 2 ]} />
  <T.MeshBasicMaterial />
  
  {#each instances as [ x, y, z, r, g, b ]}
    <Instance 
      position={[x, y, z]}
      color={[r, g, b]}
    />
  {/each}
  
</InstancedMesh>