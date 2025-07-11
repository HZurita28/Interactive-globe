<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Interactive 3D Globe</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
  </style>
</head>
<body>
  https://cdnjs.cloudflare.com/ajax/libs/three.js/r148/three.min.js
  https://cdn.jsdelivr.net/npm/three-globe@2.26.4/dist/three-globe.min.js
  https://cdn.jsdelivr.net/npm/d3@7
  <script src="https://cdn.jsdelivr.net/npm/topcript>
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const globe = new ThreeGlobe()
      .globeImageUrl('//unpkg.com/three-globe/example/img/earth-dark.jpg')
      .bumpImageUrl('//unpkg.com/three-globe/example/img/earth-topology.png')
      .showAtmosphere(true)
      .atmosphereColor('#3a228a')
      .atmosphereAltitude(0.25);

    scene.add(globe);
    camera.position.z = 300;

    // Add lights
    const ambientLight = new THREE.AmbientLight(0xbbbbbb);
    scene.add(ambientLight);
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6);
    directionalLight.position.set(1, 1, 1);
    scene.add(directionalLight);

    // Load country polygons
    fetch('https://unpkg.com/world-atlas@2/countries-110m.json')
      .then(res => res.json())
      .then(worldData => {
        const countries = topojson.feature(worldData, worldData.objects.countries).features;

        globe
          .polygonsData(countries)
          .polygonCapColor(() => `rgba(${Math.floor(Math.random()*255)},${Math.floor(Math.random()*255)},${Math.floor(Math.random()*255)},0.8)`)
          .polygonSideColor(() => 'rgba(0, 100, 0, 0.15)')
          .polygonStrokeColor(() => '#111')
          .polygonLabel(({ properties: d }) => `Country: ${d.name}`)
          .onPolygonHover(hoverD => globe
            .polygonAltitude(d => d === hoverD ? 0.12 : 0.06)
          );
      });

    // Controls
    const controls = new THREE.TrackballControls(camera, renderer.domElement);
    controls.minDistance = 150;
    controls.maxDistance = 500;

    function animate() {
      controls.update();
      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }
    animate();

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
