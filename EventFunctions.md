Event Functions
===============


A script in Unity is not like the traditional idea of a program where the code runs continuously in a loop until it completes its task. Instead, Unity passes control to a script intermittently by calling certain functions that are declared within it. Once a function has finished executing, control is passed back to Unity. These functions are known as event functions since they are activated by Unity in response to events that occur during gameplay. Unity uses a naming scheme to identify which function to call for a particular event. For example, you will already have seen the Update function (called before a frame update occurs) and the Start function (called just before the object's first frame update). Many more event functions are available in Unity; the full list can be found in the script reference page for the MonoBehaviour class along with details of their usage. The following are some of the most common and important events.


Regular Update Events
---------------------


A game is rather like an animation where the animation frames are generated on the fly. A key concept in games programming is that of making changes to position, state and behavior of objects in the game just before each frame is rendered. The [Update](ScriptRef:MonoBehaviour.Update.html) function is the main place for this kind of code in Unity. Update is called before the frame is rendered and also before animations are calculated.



````
void Update() {
	float distance = speed * Time.deltaTime * Input.GetAxis("Horizontal");
	transform.Translate(Vector3.right * distance);
}

````

The physics engine also updates in discrete time steps in a similar way to the frame rendering. A separate event function called [FixedUpdate](ScriptRef:MonoBehaviour.FixedUpdate.html) is called just before each physics update. Since the physics updates and frame updates do not occur with the same frequency, you will get more accurate results from physics code if you place it in the FixedUpdate function rather than Update.
	


````
void FixedUpdate() {
	Vector3 force = transform.forward * driveForce * Input.GetAxis("Vertical");
	rigidbody.AddForce(force);
}

````

It is also useful sometimes to be able to make additional changes at a point after the Update and FixedUpdate functions have been called for all objects in the scene and after all animations have been calculated. An example is where a camera should remain trained on a target object; the adjustment to the camera's orientation must be made after the target object has moved. Another example is where the script code should override the effect of an animation (say, to make the character's head look towards a target object in the scene). The [LateUpdate](ScriptRef:MonoBehaviour.LateUpdate.html) function can be used for these kinds of situations.



````
void LateUpdate() {
	Camera.main.transform.LookAt(target.transform);
}

````


Initialization Events
---------------------


It is often useful to be able to call initialization code in advance of any updates that occur during gameplay. The [Start](ScriptRef:MonoBehaviour.Start.html) function is called before the first frame or physics update on an object. The [Awake](ScriptRef:MonoBehaviour.Awake.html) function is called for each object in the scene at the time when the scene loads. Note that although the various objects' Start and Awake functions are called in arbitrary order, all the Awakes will have finished before the first Start is called. This means that code in a Start function can make use of other initializations previously carried out in the Awake phase.


GUI events
----------


Unity has a system for rendering GUI controls over the main action in the scene and responding to clicks on these controls. This code is handled somewhat differently from the normal frame update and so it should be placed in the [OnGUI](ScriptRef:MonoBehaviour.OnGUI.html) function, which will be called periodically.



````
void OnGUI() {
	GUI.Label(labelRect, "Game Over");
}

````

You can also detect mouse events that occur over a GameObject as it appears in the scene. This can be used for targeting weapons or displaying information about the character currently under the mouse pointer. A set of OnMouseXXX event functions (eg, [OnMouseOver](ScriptRef:MonoBehaviour.OnMouseOver.html), [OnMouseDown](ScriptRef:MonoBehaviour.OnMouseDown.html)) is available to allow a script to react to user actions with the mouse. For example, if the mouse button is pressed while the pointer is over a particular object then an OnMouseDown function in that object's script will be called if it exists.


Physics events
--------------


The physics engine will report collisions against an object by calling event functions on that object's script. The [OnCollisionEnter](ScriptRef:MonoBehaviour.OnCollisionEnter.html), [OnCollisionStay](ScriptRef:MonoBehaviour.OnCollisionStay.html) and [OnCollisionExit](ScriptRef:MonoBehaviour.OnCollisionExit.html) functions will be called as contact is made, held and broken. The corresponding [OnTriggerEnter](ScriptRef:MonoBehaviour.OnTriggerEnter.html), [OnTriggerStay](ScriptRef:MonoBehaviour.OnTriggerStay.html) and [OnTriggerExit](ScriptRef:MonoBehaviour.OnTriggerExit.html) functions will be called when the object's collider is configured as a Trigger (ie, a collider that simply detects when something enters it rather than reacting physically). These functions may be called several times in succession if more than one contact is detected during the physics update and so a parameter is passed to the function giving details of the collision (position, identity of the incoming object, etc).



````
void OnCollisionEnter(otherObj: Collision) {
	if (otherObj.tag == "Arrow") {
		ApplyDamage(10);
	}
}

````
