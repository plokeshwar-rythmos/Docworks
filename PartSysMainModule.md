#Particle System Main module

The Particle System module contains global properties that affect the whole system. Most of these properties control the initial state of newly created particles. To expand and collapse the main module, click the Particle System bar in the Inspector window.

![](../uploads/Main/PartSysPartSysInsp.png)



The name of the module appears in the inspector as the name of the GameObject that the [Particle System component](class-ParticleSystem) is attached to.

##Properties

| **Property** | **Function** |
|:---|:---| 
| __Duration__ | The length of time the system runs. |
| __Looping__ | If enabled, the system starts again at the end of its duration time and continues to repeat the cycle. |
| __Prewarm__ | If enabled, the system is initialized as though it had already completed a full cycle (only works if __Looping__ is also enabled). |
| __Start Delay__ | Delay in seconds before the system starts emitting once enabled. |
| __Start Lifetime__ | The initial lifetime for particles. |
| __Start Speed__ | The initial speed of each particle in the appropriate direction. |
| __3D Start Size__ | Enable this if you want to control the size of each axis separately. |
| __Start Size__ | The initial size of each particle. |
| __3D Start Rotation__ | Enable this if you want to control the rotation of each axis separately. |
| __Start Rotation__ | The initial rotation angle of each particle. |
| __Randomize Rotation Direction__ | Causes some particles to spin in the opposite direction. |
| __Start Color__ | The initial color of each particle. |
| __Gravity Modifier__ | Scales the gravity value set in the physics manager. A value of zero switches gravity off. |
| __Simulation Space__ | Controls whether particles are animated in the parent object’s local space (therefore moving with the parent object), in the world space, or relative to a custom object (moving with a custom object of your choosing). |
| __Simulation Speed__ | Adjust the speed at which the entire system updates. |
| __Delta Time__ | Choose between __Scaled__ and __Unscaled__, where __Scaled__ uses the __Time Scale__ value in the Time Manager, and __Unscaled__ ignores it. This is useful for Particle Systems that appear on a Pause Menu, for example. |
| __Scaling Mode__ | Choose how to use the scale from the transform. Set to __Hierarchy__, __Local__ or __Shape__. Local applies only the Particle System transform scale, ignoring any parents. Shape mode applies the scale to the start positions of the particles, but does not affect their size. |
| __Play on Awake__ | If enabled, the Particle System starts automatically when the object is created. |
| __Emitter Velocity__ | Choose how the Particle System calculates the velocity used by the Inherit Velocity and Emission modules. The system can calculate the velocity using a Rigidbody component, if one exists, or by tracking the movement of the Transform component. |
| __Max Particles__ | The maximum number of particles in the system at once. If the limit is reached, some particles are removed. |
| __Auto Random Seed__ | If enabled, the Particle System looks different each time it is played. When set to false, the system is exactly the same every time it is played. |
| __Random Seed__ | When disabling the automatic random seed, this value is used to create a unique repeatable effect. |
| __Stop Action__ | When all the particles belonging to the system have finished, it is possible to make the system perform an action. A system is determined to have stopped when all its particles have died, and its age has exceeded its Duration. For looping systems, this only happens if the system is stopped via script. |
|&nbsp;&nbsp;&nbsp;&nbsp;Disable | The GameObject is disabled. |
|&nbsp;&nbsp;&nbsp;&nbsp;Destroy | The GameObject is destroyed. |
|&nbsp;&nbsp;&nbsp;&nbsp;Callback | The OnParticleSystemStopped callback is sent to any scripts attached to the GameObject. |


##Property details

The system emits particles for a specific duration, and can be set to emit continuously using the __Looped__ property. This allows you to set particles to be emitted intermittently or continuously; for example, an object may emit smoke in short puffs or in a steady stream.

The __Start__ properties (__lifetime__, __speed__, __size__, __rotation__ and __color__) specify the state of a particle on emission. You can specify a particle’s width, height and depth independently, using the __3D Start Size__ property (see [Non-uniform particle scaling](#scaling), below).

All Particle Systems use the same gravity vector specified in the __Physics__ settings. The Gravity Multiplier value can be used to scale the gravity, or switch it off if set to zero.

The __Simulation Space__ property determines whether the particles move with the Particle System parent object, a custom object, or independently in the game world. For example, systems like clouds, hoses and flamethrowers need to be set independently of their parent GameObject, as they tend to leave trails that persist in the world space even if the object producing them moves around. On the other hand, if particles are used to create a spark between two electrodes, the particles should move along with the parent object. For more advanced control over how particles follow their Transform, see documentation on the [Inherit Velocity module](PartSysInheritVelocity).

<a name="scaling"></a>
##Non-uniform particle scaling

The 3D Start Size property allows you to specify a particle’s width, height and depth independently. In the Particle System __Main__ module, check the __3D Start Size__ checkbox, and enter the values for the initial x (width), y (height) and z (depth) of the particle. Note that z (depth) only applies to 3D Mesh particles. You can also set randomised values for these properties, in a range between two constants or curves.

You can set the particle’s initial size in the Particle System __Main__ module, and its size over the particle’s lifetime using the __Separate Axes__ option in the __Size over Lifetime__ module. You can also set the particle’s size in relation to its speed using the __Separate Axes__ option in the __Size by Speed__ module.

<br/>

-----

*  <span class="page-edit">2017-05-31  <!-- include IncludeTextAmendPageYesEdit --></span>

*  <span class="page-history">Simulation Speed, Delta Time and Emitter Velocity added in Unity [2017.1](../Manual/30_search.html?q=newin20171) <span class="search-words">NewIn20171</span></span>

*  <span class="page-history">Stop Action particle system property added in Unity [2017.2](../Manual/30_search.html?q=newin20172) <span class="search-words">NewIn20172</span></span>

