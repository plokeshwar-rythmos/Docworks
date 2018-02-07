#Physics

Unity has NVIDIA PhysX physics engine built-in. This allows for unique emergent behaviour and has many useful features.

##Basics

To put an object under physics control, simply add a __Rigidbody__ to it. When you do this, the object will be affected by gravity, and can collide with other objects in the world.

##Rigidbodies

[Rigidbodies](class-Rigidbody) are physically simulated objects. You use Rigidbodies for things that the player can push around, for example crates or loose objects, or you can move Rigidbodies around directly by adding forces to it by scripting.

If you move the Transform of a non-Kinematic Rigidbody directly it may not collide correctly with other objects. Instead you should move a Rigidbody by applying forces and torque to it. You can also add [Joints](class-HingeJoint) to rigidbodies to make the behavior more complex. For example, you could make a physical door or a crane with a swinging chain.

You also use Rigidbodies to bring vehicles to life, for example you can make cars using a Rigidbody, 4 [Wheel Colliders](class-WheelCollider) and a script applying wheel forces based on the user's [Input](class-InputManager).

You can make airplanes by applying forces to the Rigidbody from a script. Or you can create special vehicles or robots by adding various Joints and applying forces via scripting.

Rigidbodies are most often used in combination with [primitive colliders](class-BoxCollider).

**Tips:**

* You should never have a parent and child rigidbody together
* You should never scale the parent of a rigidbody


##Kinematic Rigidbodies

A __Kinematic Rigidbody__ is a Rigidbody that has the isKinematic option enabled. Kinematic Rigidbodies are not affected by forces, gravity or collisions. They are driven explicitly by setting the position and rotation of the Transform or animating them, yet they can interact with other non-Kinematic Rigidbodies.

Kinematic Rigidbodies correctly [wake up](RigidbodySleeping) other Rigidbodies when they collide with them, and they apply friction to Rigidbodies placed on top of them.

These are a few example uses for Kinematic Rigidbodies:


1. Sometimes you want an object to be under physics control but in another situation to be controlled explicitly from a script or animation. For example you could make an animated character whose bones have Rigidbodies attached that are connected with joints for use as a Ragdoll. Most of the time the character is under animation control, thus you make the Rigidbody Kinematic. But when he gets hit you want him to turn into a Ragdoll and be affected by physics. To accomplish this, you simply disable the isKinematic property.
1. Sometimes you want a moving object that can push other objects yet not be pushed itself. For example if you have an animated platform and you want to place some Rigidbody boxes on top, you should make the platform a Kinematic Rigidbody instead of just a __Collider__ without a Rigidbody.
1. You might want to have a Kinematic Rigidbody that is animated and have a real Rigidbody follow it using one of the available Joints.

##Static Colliders

A __Static Collider__ is a GameObject that has a Collider but not a Rigidbody. Static Colliders are used for level geometry which always stays at the same place and never moves around. You can add a __Mesh Collider__ to your already existing graphical meshes (even better use the __Import Settings__ Generate Colliders check box), or you can use one of the other Collider types.

Static Colliders (without a rigidbody) should not be disabled, enabled or transformed. Altering these Static Colliders will cause an internal recomputation in PhysX that is quite expensive and which will result in a big drop in performance. Even worse, this could leave the collider in an undefined state that would produce erroneous physics calculations, (for example a ray cast against an altered Static Collider could fail to detect it, or detect it at a random position in space). Also the behaviour of waking up other Rigidbodies based on a Static Collider is undefined, and moving Static Colliders will not apply friction to Rigidbodies that touch it. For these reasons only colliders that are Rigidbodies should be altered, and _isKinematic_ should be set to true for colliders that won't be affected by the physics simulation.

##Character Controllers
You use [Character Controllers](class-CharacterController) if you want to make a humanoid character. This could be the main character in a third person platformer, FPS shooter or any enemy characters.

These Controllers don't follow the rules of physics since it will not feel right (in Doom you run 90 miles per hour, come to halt in one frame and turn on a dime). Instead, a Character Controller performs collision detection to make sure your characters can slide along walls, walk up and down stairs, etc.

Character Controllers are not affected by forces but they can push Rigidbodies by applying forces to them from a script. Usually, all humanoid characters are implemented using Character Controllers.

Character Controllers are inherently unphysical, thus if you want to apply real physics - Swing on ropes, get pushed by big rocks - to your character you have to use a Rigidbody, this will let you use joints and forces on your character. Character Controllers are always aligned along the Y axis, so you also need to use a Rigidbody if your character needs to be able to change orientation in space (for example under a changing gravity). However, be aware that tuning a Rigidbody to feel right for a character is hard due to the unphysical way in which game characters are expected to behave. Another difference is that Character Controllers can slide smoothly over steps of a specified height, while Rigidbodies will not.

If you parent a Character Controller with a Rigidbody you will get a "Joint" like behavior.
