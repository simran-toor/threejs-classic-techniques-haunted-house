# 16 HAUNTED HOUSE 
* only using **Three.js primitives** as geometries and the textures in the textures folder 
## TIPS FOR MEASUREMENTS
* instead of using random measures we are going to consider **1 unit** as **1 meter**
## SETUP
* a floor
* a sphere
* some lights
* no shadow
* a Dat.GUI panel
## THE HOUSE 
* remove the sphere but leave the floor
* create the house group:
    ```
    const house = new THREE.Group()
    scene.add(house)
    ```
* create the walls:
    ```
    const walls = new THREE.Mesh(
        new THREE.BoxGeometry(4, 2.5, 4),
        new THREE.MeshStandardMaterial({ color: '#ac8e82'})
    )
    walls.position.y = 1.25
    house.add(walls)
    ```
* create the roof with a pyramid using `ConeGeometry`:
    ```
    const roof = new THREE.Mesh(
        new THREE.ConeGeometry(3.5, 1, 4),
        new THREE.MeshStandardMaterial({ color: '#b35f45'})
    )
    roof.rotation.y = Math.PI * 0.25
    roof.position.y = 2.5 + 0.5
    house.add(roof)
    ```
* create the door with a plane 
    ```
    const door = new THREE.Mesh(
        new THREE.PlaneGeometry(2, 2),
        new THREE.MeshStandardMaterial({ color: '#aa7b7b'})
    )
    door.position.y = 1 
    door.position.z = 2 + 0.01
    house.add(door)
    ```
* add bushes and use the **same geometry** and **same material** for every bush:
    ```
    const bushGeometry = new THREE.SphereGeometry(1, 16, 16)
    const bushMaterial = new THREE.MeshStandardMaterial({ color: '#89c854'})

    const bush1 = new THREE.Mesh(bushGeometry, bushMaterial)
    bush1.scale.set(0.5, 0.5, 0.5)
    bush1.position.set(0.8, 0.2, 2.2)

    const bush2 = new THREE.Mesh(bushGeometry, bushMaterial)
    bush2.scale.set(0.25, 0.25, 0.25)
    bush2.position.set(1.4, 0.1, 2.1)

    const bush3 = new THREE.Mesh(bushGeometry, bushMaterial)
    bush3.scale.set(0.4, 0.4, 0.4)
    bush3.position.set(-0.8, 0.1, 2.2)

    const bush4 = new THREE.Mesh(bushGeometry, bushMaterial)
    bush4.scale.set(0.15, 0.15, 0.15)
    bush4.position.set(-1, 0.05, 2.6)

    house.add(bush1, bush2, bush3, bush4)
    ```
## THE GRAVES 
* instead of placing each grave manually. going to create and place them procedurally
* create a graves container:
    ```
    const graves = new THREE.Group()
    scene.add(graves)
    ```
* create one `BoxGeometry` and one `MeshStandardMaterial` that will be used for every grave mesh:
    ```
    const graveGeometry = new THREE.BoxGeometry(0.6, 0.8, 0.2)
    const graveMaterial = new THREE.MeshStandardMaterial({ color: '#b2b6b1'})
    ```
* create the graves around the house:
    ```
    const graves = new THREE.Group()
    scene.add(graves)

    const graveGeometry = new THREE.BoxGeometry(0.6, 0.8, 0.2)
    const graveMaterial = new THREE.MeshStandardMaterial({ color: '#b2b6b1'})

    for(let i =0; i < 50; i++){
        const angle = Math.random() * Math.PI * 2 
        const radius = 3 + Math.random() * 6      
        const x = Math.sin(angle) * radius        
        const z = Math.cos(angle) * radius

        const grave = new THREE.Mesh(graveGeometry, graveMaterial)
        grave.position.set(x, 0.3, z)
        grave.rotation.z = (Math.random() - 0.5 ) * 0.4
        grave.rotation.y = (Math.random() - 0.5) * 0.4
        
        graves.add(grave)
    }
    ```
## LIGHTS
* dim the ambient and moon lights and give those a more blue-ish colour:
    ```
    const ambientLight = new THREE.AmbientLight('#b9d5ff', 0.12)


    // ...
    const moonLight = new THREE.DirectionalLight('#b9d5ff', 0.12)
    ```
* add a warm `PointLight` above the door and add it to the **house** instead of **scene**:
    ```
    const doorLight = new THREE.PointLight('#ff7d46', 1, 7)
    doorLight.position.set(0, 2.2, 2.7)
    house.add(doorLight)
    ```
## FOG 
* Three.js supports fog with the `Fog` class
    - `color`
    - `near` - how far from the camera fog starts
    - `far` - how far from the camera the fog is fully opaque
* instantiate after `// Scene`:
    ```
    const fog = new THREE.Fog('#262837', 1, 15)
    ```
* to activate, add to scene:
    ```
    scene.fog = fog 
    ```
* to fix the background, need to change the clear colour of the `renderer` and use the same colour as the fog
* use `setClearColor(...)` on the **renderer**:
    ```
    renderer.setClearColor('#262837')
    ```
## TEXTURES
### THE DOOR 
* load all the door textures:
    ```
    const doorColorTexture = textureLoader.load('/textures/door/color.jpg')
    const doorAlphaTexture = textureLoader.load('/textures/door/alpha.jpg')
    const doorAmbientOcclusionTexture = textureLoader.load('/textures/door/ambientOcclusion.jpg')
    const doorHeightTexture = textureLoader.load('/textures/door/height.jpg')
    const doorNormalTexture = textureLoader.load('/textures/door/normal.jpg')
    const doorMetalnessTexture = textureLoader.load('/textures/door/metalness.jpg')
    const doorRoughnessTexture = textureLoader.load('/textures/door/roughness.jpg')
    ```
* apply them to the `door`, add subdivisions to the geometry and inclue the `uv2` attribute to support the `aoMap`:
    ```
    const door = new THREE.Mesh(
        new THREE.PlaneGeometry(2, 2),
        new THREE.MeshStandardMaterial({
            map: doorColorTexture, 
            transparent: true,

            //needs `transparent: true` before it
            alphaMap: doorAlphaTexture,  

            //need `uv` coords (use same as current), done outside function 
            aoMap: doorAmbientOcclusionTexture,  

            //needs more vertices (play with subdivisions, 3rd&4th params)
            displacementMap: doorHeightTexture,
            displacementScale: 01, 
            normalMap: doorNormalTexture,
            metalnessMap: doorMetalnessTexture,
            roughnessMap: doorRoughnessTexture
        })
    )
    door.geometry.setAttributes(
        'uv2',
        new THREE.Float32BufferAttribute(door.geometry.attributes.uv.array, 2))
    ```
* door is slightly floating above the ground, so increase the size:
    ```
    new THREE.PlaneGeometry(2.2, 2.2, 100, 100),
    ```
### THE WALLS 
* load brick textures:
```
const bricksColorTexture = textureLoader.load('/textures/bricks/color.jpg')
const bricksAmbientOcclusionTexture = textureLoader.load('/textures/bricks/ambientOcclusion.jpg')
const bricksNormalTexture = textureLoader.load('/textures/bricks/normal.jpg')
const bricksRoughnessTexture = textureLoader.load('/textures/bricks/roughness.jpg')
```
* apply them to the `wall`:
```
const walls = new THREE.Mesh(
    new THREE.BoxGeometry(4, 2.5, 4),
    new THREE.MeshStandardMaterial({
        map: bricksColorTexture, 
        aoMap: bricksAmbientOcclusionTexture,
        normalMap: bricksNormalTexture,
        roughnessMap: bricksRoughnessTexture
    })
)
```
### THE FLOOR
* load the grass textures:
    ```
    const grassColorTexture = textureLoader.load('/textures/grass/color.jpg')
    const grassAmbientOcclusionTexture = textureLoader.load('/textures/grass/ambientOcclusion.jpg')
    const grassNormalTexture = textureLoader.load('/textures/grass/normal.jpg')
    const grassRoughnessTexture = textureLoader.load('/textures/grass/roughness.jpg')
    ```
* apply to floor: 
    ```
    const floor = new THREE.Mesh(
        new THREE.PlaneGeometry(20, 20),
        new THREE.MeshStandardMaterial({
            map: grassColorTexture,
            aoMap: grassAmbientOcclusionTexture,
            normalMap: grassNormalTexture,
            roughnessMap: grassRoughnessTexture
        })
    )
    floor.geometry.setAttribute(
        'uv2',
        new THREE.Float32BufferAttribute(floor.geometry.attributes.uv.array, 2)
    )
    ```
* play with the repeat of the grass texture:
    ```
    grassColorTexture.repeat.set(8, 8)
    grassAmbientOcclusionTexture.repeat.set(8, 8)
    grassNormalTexture.repeat.set(8, 8)
    grassRoughnessTexture.repeat.set(8, 8)
    ```
* by default the texture doesn't repeat
* change the `wrapS` and `wrapT` properties to activate the repeat:
    ```
    grassColorTexture.wrapS = THREE.RepeatWrapping
    grassAmbientOcclusionTexture.wrapS = THREE.RepeatWrapping
    grassNormalTexture.wrapS = THREE.RepeatWrapping
    grassRoughnessTexture.wrapS = THREE.RepeatWrapping

    grassColorTexture.wrapT = THREE.RepeatWrapping
    grassAmbientOcclusionTexture.wrapT = THREE.RepeatWrapping
    grassNormalTexture.wrapT = THREE.RepeatWrapping
    grassRoughnessTexture.wrapT = THREE.RepeatWrapping
    ```
## GHOSTS
* creating ghosts which are represented by simple lights floating around the house and passing through the ground and graves:
    ```
    const ghost1 = new THREE.PointLight('#ff00ff', 2, 3)
    scene.add(ghost1)

    const ghost2 = new THREE.PointLight('#00ffff', 2, 3)
    scene.add(ghost2)

    const ghost3 = new THREE.PointLight('#ffff00', 2, 3)
    scene.add(ghost3)
    ```
* Animate them:
    ```
    const ghost1Angle = elapsedTime * 0.5
    ghost1.position.x = Math.cos(ghost1Angle) * 4
    ghost1.position.z = Math.sin(ghost1Angle) * 4
    ghost1.position.y = Math.sin(elapsedTime * 3)

    const ghost2Angle = elapsedTime * 0.32
    ghost2.position.x = Math.cos(ghost2Angle) * 5
    ghost2.position.z = Math.sin(ghost2Angle) * 5
    ghost2.position.y = Math.sin(elapsedTime * 4) + Math.sin(elapsedTime * 2.5)

    const ghost3Angle = elapsedTime * 0.18
    ghost3.position.x = Math.cos(ghost3Angle) * (7 + Math.sin(elapsedTime * 0.32))
    ghost3.position.z = Math.sin(ghost3Angle) * (7 + Math.sin(elapsedTime * 0.5))
    ghost3.position.y = Math.sin(elapsedTime * 4) + Math.sin(elapsedTime * 2.5)
    ```
## SHADOWS
* activate the shadow map on the renderer:
```
renderer.shadowMap.enabled = true
```
* go through each light and activate the shadows on the lights that should cast shadows:
    ```
    moonLight.castShadow = true
    doorLight.castShadow = true
    ghost1.castShadow = true
    ghost2.castShadow = true
    ghost3.castShadow = true
    ```
* go through each object in scene and decide if it can cast and/or receive shadows:
    ```
    walls.castShadow = true
    bush1.castShadow = true
    bush2.castShadow = true
    bush3.castShadow = true
    bush4.castShadow = true 

    floor.receiveShadow = true

    // in for loop for graves
    graves.castShadow = true
    ```
* optimise shadow maps:
    ```
    doorLight.shadow.mapSize.width = 256
    doorLight.shadow.mapSize.height = 256
    doorLight.shadow.camera.far = 7

    ghost1.shadow.mapSize.width = 256
    ghost1.shadow.mapSize.height = 256
    ghost1.shadow.camera.far = 7

    ghost2.shadow.mapSize.width = 256
    ghost2.shadow.mapSize.height = 256
    ghost2.shadow.camera.far = 7

    ghost3.shadow.mapSize.width = 256
    ghost3.shadow.mapSize.height = 256
    ghost3.shadow.camera.far = 7
    ```
* change algorithm for shadow maps:
    ```
    renderer.shadowMap.type = THREE.PCFSoftShadowMap
    ```
