# clmg
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quadrilateral Area Calculator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three/examples/js/controls/OrbitControls.js"></script>
    <style>
        body { margin: 0; font-family: Arial, sans-serif; }
        canvas { display: block; }
        
        #controlPanel {
            position: absolute;
            top: 10px;
            left: 10px;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 15px;
            border-radius: 5px;
            font-size: 16px;
        }

        #controlPanel label {
            display: block;
            margin-top: 8px;
        }

        #controlPanel input {
            width: 80px;
            padding: 5px;
            font-size: 14px;
        }

        #areaDisplay {
            margin-top: 10px;
            font-size: 18px;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <!-- Control Panel for Customization -->
    <div id="controlPanel">
        <label>AB: <input type="number" id="sideAB" value="3" min="1"></label>
        <label>BC: <input type="number" id="sideBC" value="8" min="1"></label>
        <label>CD: <input type="number" id="sideCD" value="5" min="1"></label>
        <label>∠ABC (degrees): <input type="number" id="angleABC" value="60" min="1" max="180"></label>
        <div id="areaDisplay">Area: Calculating...</div>
    </div>

    <script>
        function calculateArea() {
            let AB = parseFloat(document.getElementById("sideAB").value);
            let BC = parseFloat(document.getElementById("sideBC").value);
            let CD = parseFloat(document.getElementById("sideCD").value);
            let angleABC = parseFloat(document.getElementById("angleABC").value);

            // Convert angle to radians
            let angleABC_radians = angleABC * Math.PI / 180;

            // Law of Cosines to find AC
            let AC = Math.sqrt(
                AB**2 + BC**2 - 2 * AB * BC * Math.cos(angleABC_radians)
            );

            // Sine Rule to find angle BCD
            let angleBCD_radians = Math.asin((BC * Math.sin(angleABC_radians)) / AC);
            let angleBCD = angleBCD_radians * (180 / Math.PI); // Convert back to degrees

            // Triangle ABC using Heron's formula
            let s1 = (AB + BC + AC) / 2;
            let areaABC = Math.sqrt(s1 * (s1 - AB) * (s1 - BC) * (s1 - AC));

            // Triangle BCD using Heron's formula
            let s2 = (BC + CD + AC) / 2;
            let areaBCD = Math.sqrt(s2 * (s2 - BC) * (s2 - CD) * (s2 - AC));

            // Total Quadrilateral Area
            let quadrilateralArea = areaABC + areaBCD;

            // Display result
            document.getElementById("areaDisplay").innerText = 
                `Quadrilateral Area ≈ ${quadrilateralArea.toFixed(2)}
                \nDiagonal AC: ${AC.toFixed(2)}
                \n∠BCD: ${angleBCD.toFixed(2)}°`;
        }

        // Update area whenever input values change
        document.getElementById("sideAB").addEventListener("input", calculateArea);
        document.getElementById("sideBC").addEventListener("input", calculateArea);
        document.getElementById("sideCD").addEventListener("input", calculateArea);
        document.getElementById("angleABC").addEventListener("input", calculateArea);

        // Initial calculation
        calculateArea();
    </script>

</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Draggable Cyclic Quadrilateral</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three/examples/js/controls/OrbitControls.js"></script>
    <style>
        body { margin: 0; font-family: Arial, sans-serif; }
        canvas { display: block; }
    </style>
</head>
<body>
    <script>
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 0, 20);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;

        // Circle outline
        const radius = 7;
        const circleGeometry = new THREE.BufferGeometry();
        const circleVertices = [];
        for (let i = 0; i <= 100; i++) {
            let angle = (i / 100) * Math.PI * 2;
            circleVertices.push(radius * Math.cos(angle), radius * Math.sin(angle), 0);
        }
        circleGeometry.setAttribute('position', new THREE.Float32BufferAttribute(circleVertices, 3));
        const circleMaterial = new THREE.LineBasicMaterial({ color: 0x3399ff });
        const circle = new THREE.LineLoop(circleGeometry, circleMaterial);
        scene.add(circle);

        function polarToCartesian(r, angle) {
            return new THREE.Vector3(r * Math.cos(angle), r * Math.sin(angle), 0);
        }

        let angles = [Math.PI / 4, Math.PI / 2, (3 * Math.PI) / 4, Math.PI];
        let labels = ["A", "B", "C", "D"];
        let markers = [];

        // Function to create text labels
        function createLabel(text) {
            const canvas = document.createElement("canvas");
            const ctx = canvas.getContext("2d");
            canvas.width = 64;
            canvas.height = 64;
            ctx.fillStyle = "white";
            ctx.font = "32px Arial";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText(text, 32, 32);

            const texture = new THREE.CanvasTexture(canvas);
            const spriteMaterial = new THREE.SpriteMaterial({ map: texture });
            const sprite = new THREE.Sprite(spriteMaterial);
            sprite.scale.set(1.5, 1.5, 1);
            return sprite;
        }

        angles.forEach((angle, index) => {
            let position = polarToCartesian(radius, angle);

            // Create the yellow dots
            const markerGeometry = new THREE.SphereGeometry(0.3, 16, 16);
            const markerMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 });
            const marker = new THREE.Mesh(markerGeometry, markerMaterial);
            marker.position.copy(position);
            scene.add(marker);

            // Create labels A, B, C, D
            const label = createLabel(labels[index]);
            label.position.copy(position.clone().add(new THREE.Vector3(0.5, 0.5, 0))); // Offset label slightly
            scene.add(label);

            markers.push({ marker, label, angle });
        });

        // Center of the circle (O)
        const centerGeometry = new THREE.SphereGeometry(0.3, 16, 16);
        const centerMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff });
        const center = new THREE.Mesh(centerGeometry, centerMaterial);
        center.position.set(0, 0, 0);
        scene.add(center);

        // Label for center (O)
        const centerLabel = createLabel("O");
        centerLabel.position.set(0.5, 0.5, 0);
        scene.add(centerLabel);

        // Line connecting all four points
        const lineGeometry = new THREE.BufferGeometry();
        const lineMaterial = new THREE.LineBasicMaterial({ color: 0xffffff });
        const line = new THREE.Line(lineGeometry, lineMaterial);
        scene.add(line);

        // Line connecting A and C
        const acLineGeometry = new THREE.BufferGeometry();
        const acLineMaterial = new THREE.LineBasicMaterial({ color: 0xff0000 });
        const acLine = new THREE.Line(acLineGeometry, acLineMaterial);
        scene.add(acLine);

        function updateLines() {
            const positions = [];
            markers.forEach(m => {
                positions.push(m.marker.position.x, m.marker.position.y, m.marker.position.z);
            });
            positions.push(markers[0].marker.position.x, markers[0].marker.position.y, markers[0].marker.position.z); // Close the shape
            lineGeometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
            lineGeometry.attributes.position.needsUpdate = true;

            // Update labels' positions
            markers.forEach(m => {
                m.label.position.copy(m.marker.position.clone().add(new THREE.Vector3(0.5, 0.5, 0)));
            });

            // Update AC line positions
            const acPositions = [
                markers[0].marker.position.x, markers[0].marker.position.y, markers[0].marker.position.z, // A
                markers[2].marker.position.x, markers[2].marker.position.y, markers[2].marker.position.z  // C
            ];
            acLineGeometry.setAttribute('position', new THREE.Float32BufferAttribute(acPositions, 3));
            acLineGeometry.attributes.position.needsUpdate = true;
        }
        updateLines();

        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();
        let draggedMarker = null;

        window.addEventListener("mousedown", (event) => {
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(markers.map(m => m.marker));
            if (intersects.length > 0) {
                draggedMarker = intersects[0].object;
            }
        });

        window.addEventListener("mousemove", (event) => {
            if (!draggedMarker) return;
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
            raycaster.setFromCamera(mouse, camera);
            let intersection = raycaster.ray.intersectPlane(new THREE.Plane(new THREE.Vector3(0, 0, 1), 0));
            if (intersection) {
                let angle = Math.atan2(intersection.y, intersection.x);
                draggedMarker.position.copy(polarToCartesian(radius, angle));
                let markerObj = markers.find(m => m.marker === draggedMarker);
                if (markerObj) markerObj.angle = angle;
                updateLines();
            }
        });

        window.addEventListener("mouseup", () => {
            draggedMarker = null;
        });

        function animate() {
            requestAnimationFrame(animate);
            controls.update();
            renderer.render(scene, camera);
        }
        
        animate();
    </script>
</body>
</html>
