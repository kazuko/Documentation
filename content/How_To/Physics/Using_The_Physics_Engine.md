---
PG_TITLE: How to Use a Physics Engine
---

# How to Use a Physics Engine

## Introduction

Babylon.js has a plugin system for physics engines that enables the user to add physics interactions to the scene's objects.
Unlike the internal collision system, a physics engine calculates objects'  body dynamics and emulates "real-life" interactions between them. So if two objects collide, they will "bounce" off one another, just like you would expect from a real-life object.

Babylon.js' plugin system allowed us to use well established physics engines and to integrate them into Babylon.js' render loop. Apart from very advanced usage, there is no need to interact directly with the physics engine. Babylon.js does the work for you.

This tutorial will show the basic usage of the physics system.

## What physics engine are integrated

There are plugins for 3 physics engines:

1. Cannon.js - a wonderful physics engine written entirely in JavaScript
1. Oimo.js - a JS port of the lightweight Oimo physics engine
1. Energy.js - (Not yet publicly available) - a JS port of a C++ physics engine

Each engine has its own features and its own way of calculating the body dynamics. We at Babylon.js tried collecting the common usage of all engines and provide an easy-to-use interface to them.

Once you picked an engine, do not forget to add a reference to the engine code:

1. Cannon: https://cdn.babylonjs.com/cannon.js
1. Oimo: https://cdn.babylonjs.com/Oimo.js

## Basic usage

### Enabling the physics engine

To enable the physics engine, call the scene's `enablePhysics` function:

```javascript
var scene = new BABYLON.Scene(engine);
var gravityVector = new BABYLON.Vector3(0,-9.81, 0);
var physicsPlugin = new BABYLON.CannonJSPlugin();
scene.enablePhysics(gravityVector, physicsPlugin);
```

Both parameters are optional. The default parameters are shown in the example. This is the same as calling:

```javascript
scene.enablePhysics();
```

To use OimoJS simply change the 2nd parameter to `new BABYLON.OimoJSPlugin()`:

```javascript
scene.enablePhysics(new BABYLON.Vector3(0,-9.81, 0), new BABYLON.OimoJSPlugin());
```

Calling this function will create a new BABYLON.PhysicsEngine object that will be in charge of handling the physics interactions.

The physics engine is now enabled and is running during the render loop.

### Impostors

To allow interaction between objects, the physics engines use an impostor, which is a simpler representation of a complex object. 
An impostor, as a rule, is a rigid body - meaning it cannot be changed during interaction. A sphere will always have the same radius, a box will always have the same length. If you want to change the object, a new impostor will be created.

Each physics engine has different types of Impostors. The following table shows what each engine supports, and what it uses to simulate the missing impostors

| Impostor Type | Cannon.js | Oimo.js | Energy.js | Notes   |
|---------------|-----------|---------|-----------|---------|
| Box           | Box       | Box     | Box       |         |
| Sphere        | Sphere    | Sphere  | Sphere    |         |
| Particle      | Particle  | Sphere  | Unknown   |         |
| Plane         | Plane     | Box     | Plane     | Simulates an unlimited surface. Like a floor that never ends. Consider using Box |
| Cylinder      | Cylinder  | Cylinder| Cylinder  |         |
| Mesh          | Mesh      | Box     | Mesh      | Use only when necessary - will lower performance. Cannon's mesh impostor only collides against spheres and planes |
| Heightmap     | Heightmap | Box     | Mesh      |         |

Using simple impostors for complex objects will increase performance but decrease the reality of the scene's physics. Consider when complex impostors (like the mesh or the heightmap) is needed, and when the simpler geometries can be used.

### Babylon's physics impostor

To enable physics on an object(*) you need to assign it a physics impostor. The signature of the impostor's constructor is (provided with TypeScript type definition):

```javascript
new BABYLON.PhysicsImpostor(object: IPhysicsEnabledObject, type: number, options: PhysicsImpostorParameters, scene:BABYLON.Scene);
```

#### object

You will notice that I keep on writing object and not mesh, and that the first parameter is not a mesh but an interface (IPhysicsEnabledObject). It is possible to assign an impostor to any Babylon object that has at least two parameters:

```javascript
position: BABYLON.Vector3;
rotationQuaternion: BABYLON.Quaternion
```

An AbstractMesh will be the first choice, of course. But a Solid Particle also applies, and so does a light or certain cameras. I will show how to use an impostor on different object types in the advanced tutorial.

#### type

Type can be one of the following:

```javascript
BABYLON.PhysicsImpostor.SphereImpostor;
BABYLON.PhysicsImpostor.BoxImpostor;
BABYLON.PhysicsImpostor.PlaneImpostor;
BABYLON.PhysicsImpostor.MeshImpostor;
BABYLON.PhysicsImpostor.CylinderImpostor;
BABYLON.PhysicsImpostor.ParticleImpostor;
BABYLON.PhysicsImpostor.HeightmapImpostor;
```

#### options

Options is a JSON. The interface is as follows:

```javascript
    export interface PhysicsImpostorParameters {
        mass: number;
        friction?: number;
        restitution?: number;
        nativeOptions?: any;
        ignoreParent?: boolean;
        disableBidirectionalTransformation?: boolean;
    }
```

* mass: The only mandatory parameters is mass, which is the object's mass in kg. A `0` as a value will create a static impostor - good for floors.
* friction: is the impostor's friction when colliding against other impostors.
* restitution: is the amount of force the body will "give back" when colliding. A low value will create no bounce, a value of 1 will be a very bouncy interaction.
* nativeOptions: is a JSON with native options of the selected physics plugin. More about it in the advanced tutorial.
* ignoreParent: when using babylon's parenting system, the physics engine will use the compound system. To avoid using the compound system, set this flag to true. More about it in the advanced tutorial.
* disableBidirectionalTransformation: will disable the bidirectional transformation update. Setting this will make sure the physics engine ignores changes made to the mesh's position and rotation (and will increase performance a bit)

#### scene

I hope no explanation is required.

### Basic physics scene

I will extend the playground's basic scene to have physics interactions between the sphere and the ground.

I will first have to enable physics:

```javascript
scene.enablePhysics();
```

Afterwards, I can create the impostors.

```javascript
sphere.physicsImpostor = new BABYLON.PhysicsImpostor(sphere, BABYLON.PhysicsImpostor.SphereImpostor, { mass: 1, restitution: 0.9 }, scene);
ground.physicsImpostor = new BABYLON.PhysicsImpostor(ground, BABYLON.PhysicsImpostor.BoxImpostor, { mass: 0, restitution: 0.9 }, scene);
```

Playground example:  https://www.babylonjs-playground.com/#BEFOO

### Further functionality of the Impostor class

In the example above, you noticed I kept a reference of the physics impostor attached to the sphere and the ground. This is not mandatory, but it is recommended to keep a reference of this object in order to interact with the physics body.

The physics impostor holds a set of functions that can be executed on the physics engine's body:

#### Bidirectional transformation linking

The physics impostor synchronizes the physics engine's body and the connected object with each frame.
That means that changing the object's position or rotation in Babylon code will also move the impostor. The impostor is also the one updating the object's position after the physics engine is finished calculating the next step.

Playground example (sphere rotation and position) -  https://www.babylonjs-playground.com/#B5BDU
Notice how the sphere rotates (due to the rotate function), but this rotation is not being taken into account by the physics engine.

Playground example (box rotation and position) -  https://www.babylonjs-playground.com/#2ADVLV
In this case, the rotation does influence the physics engine due to the geometric shape - a box standing on its edge will need to fall to either side, which influences its velocities.

#### Linear velocity

Simply put, the linear velocity is in charge of updating the object's position. A velocity in any axis will cause a movement in its direction.
To get the object's liner velocity (a BABYLON.Vector3):

```javascript
impostor.getLinearVelocity();
```

To set the object's linear velocity use:

```javascript
impostor.setLinearVelocity(new BABYLON.Vector3(0,1,0));
```

Playground example -  https://www.babylonjs-playground.com/#BXII

The physics engine is in charge of calculating the body's velocity. Changing it will not make it fixed, but give it a "push". The physics engine will take the velocity into account and will modify it using gravity and collision interactions.

#### Angular velocity

If the linear velocity was changing the position, the angular velocity is changing the rotation.

To get the object's angular velocity (a BABYLON.Quaternion):

```javascript
impostor.getAngularVelocity();
```

To set the object's linear velocity use:
```javascript
impostor.setAngularVelocity(new BABYLON.Quaternion(0,1,0,0));
```

Playground example -  https://www.babylonjs-playground.com/#IGM3H

Same as the linear velocity - setting this value will only cause the physics engine to recalculate the body dynamics. The value will not stay fixed.

#### Impulses and forces

Applying a force/impulse on a body will change its velocities (linear and angular) according to the body's properties (mass is taken into account, for example).

Cannon supports both force and impulse (different aspects of the same concept. Read about the difference here - http://www.differencebetween.com/difference-between-impulse-and-vs-force/)
Oimo only supports impulses. Applying a force will fallback to impulse.

To apply an impulse, use the applyImpulse function of the impostor:

```javascript
impostor.applyImpulse(new BABYLON.Vector3(10, 10, 0), sphere.getAbsolutePosition());
```

The first variable is the direction and amount of impulse to apply. The second is where on the body itself the force will be applied. Using this in a game of pool - you can hit the ball at various contact point locations and the interaction will vary (sometimes called "using English"). This is the way to simulate that.

Playground example -  https://www.babylonjs-playground.com/#26LQEZ
Playground example with a different position of the impulse, giving the ball a "spin" -  https://www.babylonjs-playground.com/#26LQEZ#1

#### Radial explosion impulse/force & gravitational fields

You have the ability to create radial explosions & gravitational forces. 

The forces are never applied to impostors that have mass equal 0 (the ground for example).

```javascript
var physicsHelper = new BABYLON.PhysicsHelper(scene);

var origin = BABYLON.Vector3(0, 0, 0);
var radius = 10;
var strength = 20;
var falloff = BABYLON.PhysicsRadialImpulseFalloff.Linear; // or BABYLON.PhysicsRadialImpulseFalloff.Constant

var explosionEvent = physicsHelper.applyRadialExplosionImpulse( // or .applyRadialExplosionForce
    origin,
    radius,
    strength,
    falloff
);

// or

var gravitationalFieldEvent = physicsHelper.gravitationalField(
    origin,
    radius,
    strength,
    falloff
);
gravitationalFieldEvent.enable(); // need to call, if you want to activate the gravitational field.
setTimeout(function (gravitationalFieldEvent) { gravitationalFieldEvent.disable(); }, 3000, gravitationalFieldEvent);

// or

var updraftEvent = physicsHelper.updraft(
    origin,
    radius,
    strength,
    height,
    BABYLON.PhysicsUpdraftMode.Center // or BABYLON.PhysicsUpdraftMode.Perpendicular
);
updraftEvent.enable();
setTimeout(function (updraftEvent) { updraftEvent.disable(); }, 5000, updraftEvent);
```

In case you want to do some debug, like visually show the sphere and/or rays, you can do that by calling `event.getData()` *(note that if you do that, you will need to manually call `event.dispose()` to dispose the unused meshes, after you are done debugging)*. The `event.getData()` will return back the `sphere` mesh variable, which you can then use, to apply a semi-transparent material, so you can visualize it. The `explosionEvent.getData()` will also return back the `rays` rays variable, in case you want them for debugging purposes.

*For a more detailed explanation, please take a look at the playground example below.*

Playground example -  https://playground.babylonjs.com/index.html#0LM7CJ#6

Playground example - Updraft -  https://playground.babylonjs.com/index.html#TVUDC1#3

#### Collision callbacks

You can add a callback function that will be called when an impostor collides with another impostor. 
This is how to change the color of an object if it collides against the ground.

```javascript
sphereImpostor.registerOnPhysicsCollide(groundImpostor, function(main, collided) {
    main.object.material.diffuseColor = new BABYLON.Color3(Math.random(), Math.random(), Math.random());
});
```

Note that in this case, I assumed the impostor's body is a mesh with a material.

Playground example -  https://www.babylonjs-playground.com/#1NASOD

Notice that the callback will be executed each and every time both impostors collide, but will stop when they are touching (when the sphere no longer bounces).

### Physics Joints

#### What are joints

To connect two impostors together, you can now use joints.
Think of the joint as a limitation (or constraint) of either rotation or position (or both) between two impostors.
Each engine supports different types of joints (which usually have different names as well):

| Joint Type | Cannon.js | Oimo.js | Energy.js | Notes   |
|---------------|-----------|---------|-----------|---------|
| Distance  | Distance | Distance | ---   |  A fixed distance between two impostors |
| Hinge | Hinge | Hinge | Hinge | A joint allowing rotation on a single axis (much like your knee) |
| Hinge2| ----  | Wheel  | Hinge2   | A joint allowing rotation on a single axis in two different points |
| Ball And Socket | Point To Point | Ball | Ball And Socket | A joint allowing one of the objects to rotate around a specific socket (like your hip) |
| Slider | ---- | Slider | Slider | A joint allowing changing the position along a single axis |

Cannon also has a special Spring joint that will simulate a spring connected between two impostors.

#### Adding a new joint

To add a new joint the impostor has two help classes:

```javascript
impostor.addJoint(otherImpostor, joint);
//or
impostor.createJoint(otherImpostor, jointType, jointData);
```

Joint types can be selected from the following enum:

```javascript
BABYLON.PhysicsJoint.DistanceJoint;
BABYLON.PhysicsJoint.HingeJoint;
BABYLON.PhysicsJoint.BallAndSocketJoint;
BABYLON.PhysicsJoint.WheelJoint;
BABYLON.PhysicsJoint.SliderJoint;
BABYLON.PhysicsJoint.Hinge2Joint = BABYLON.PhysicsJoint.WheelJoint;
BABYLON.PhysicsJoint.PointToPointJoint = BABYLON.PhysicsJoint.BallAndSocketJoint;
BABYLON.PhysicsJoint.SpringJoint;
```

Babylon has 3 help-classes to add joints:

`BABYLON.DistanceJoint` , `BABYLON.HingeJoint`, `BABYLON.Hinge2Joint`.

DistanceJoint playground -  https://www.babylonjs-playground.com/#26QVLZ 
SpringJoint example -  https://www.babylonjs-playground.com/#1BHF6C

#### Setting the joint data

The physics joint data interface looks like this:

```javascript
interface PhysicsJointData {
    mainPivot?: Vector3;
    connectedPivot?: Vector3;
    mainAxis?: Vector3,
    connectedAxis?: Vector3,
    collision?: boolean
    nativeParams?: any; //Native Oimo/Cannon data
}
```

* mainPivot: is the point on the main mesh (the mesh creating the joint) to which the constraint will be connected. Demo: http://www.babylonjs-playground.com/#BGUY#3
* connectedPivot: is the point on the connected mesh (the mesh creating the joint) to which the constraint will be connected.
* mainAxis: the axis on the main object on which the constraint will work. http://www.babylonjs-playground.com/#BGUY#5
* connectedAxis: the axis on the connected object on which the constraint will work.
* collision: should the two connected objects also collide with each other. The objects are sometimes forced to be close by and this can prevent constant collisions between them.
* nativParams: further parameters that will be delivered to the constraint without a filter. Those are native parameters of the specific physics engine you chose.

You can read further about joint data in this blog article : https://blog.raananweber.com/2016/09/06/webgl-car-physics-using-babylon-js-and-oimo-js/

### Interaction with the physics engine

Using `scene.getPhysicsEngine()`, you can get access to functions that will influence the engine directly.

#### Setting the time step

The physics engine assumes a certain frame-rate to be taken into account when calculating the interactions.
The time between each step can be changed to "accelerate" or "slow down" the physics interaction.
Here is the same scene with different time steps - accelerating and slowing down:

Default time step -  https://www.babylonjs-playground.com/#2B84TV
Slowing down -  https://www.babylonjs-playground.com/#2B84TV#1
Speeding up -  https://www.babylonjs-playground.com/#2B84TV#2

#### Setting the scene's gravity

You can change the scene's gravity using the physics engine's `setGravity(vector3)` function.
This can be done in real time, even after setting the gravity:

Playground demo (click to toggle positive/negative gravity) -  https://www.babylonjs-playground.com/#A2WGF
