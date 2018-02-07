#iOS Game Controller support

Starting with OS 7, a standardized Game Controller Input API is provided by Apple. Unity support for this API comes as part of the standard Unity Input API.

##Detecting attached Game Controllers
Calling `Input.GetJoystickNames` will enumerate the names of all attached controllers. Names follow the pattern: "[$profile_type,$connection_type] joystick $number by $model". $profile_type might be either "basic" or "extended", $connection_type is either "wired" or "wireless". It could be used to detect the kind of connected controller. It is worth re-checking this list every few seconds to detect if controllers have been attached or detached. Here’s an example how of to do it in C#:

````
private bool connected = false;

IEnumerator CheckForControllers() {
	while (true) {
		var controllers = Input.GetJoystickNames();
		if (!connected && controllers.Length > 0) {
			connected = true;
			Debug.Log("Connected");
		} else if (connected && controllers.Length == 0) {
			connected = false;
			Debug.Log("Disconnected");
		}
		yield return new WaitForSeconds(1f);
	}
}

void Awake() {
	StartCoroutine(CheckForControllers());
}
````

When controllers are detected you can either hide on-screen touch controls or amend them to supplement controller input. The next task is to check for Game Controller input.

##Handling input
Actual input scheme is highly dependent on the type of game you are developing. But in any case it goes down to reading axes and button states. Take following 2D game stage as simple example:


The player controls the bean character, which can move right or left, jump and fire at the bad guys. By default, the Unity Input "Horizontal" axis is mapped to basic profile game controller dpad and the left analog stick on extended profile controllers. So the code used to move the character back and forth is pretty simple:

````
float h = Input.GetAxis("Horizontal");
if (h * rigidbody2D.velocity.x < maxSpeed)
	rigidbody2D.AddForce(Vector2.right * h * moveForce);
````

You can set up jump and fire actions in Unity's Input Manager. 

* Access it from the Unity editor menu as follows: Edit > Project Settings > Input. 

* Pick joystick button "A" for the "Jump" action and "X" for "Fire". 

* Open the following actions in the Unity Input Manager and specify "joystick button 14" for "Jump" and "joystick button 15" for "Fire".


The code handling then looks like this:

````
if (Input.GetButtonDown("Jump") && grounded) {
	rigidbody2D.AddForce(new Vector2(0f, jumpForce));
}

if (Input.GetButtonDown("Fire")) {
	Rigidbody2D bulletInstance = Instantiate(rocket, transform.position, Quaternion.Euler(new Vector3(0,0,0))) as Rigidbody2D;
	bulletInstance.velocity = new Vector2(speed, 0);
}
````

The following cheatsheet should help you map controller input in the Unity Input Manager:

|Name |KeyCode |Axis |
|:---|:---|:---|
|A |joystick button 14 |joystick axis 14 |
|B |joystick button 13 |joystick axis 13 |
|X |joystick button 15 |joystick axis 15 |
|Y |joystick button 12 |joystick axis 12 |
|Left Stick |N/A |Axis 1 (X) - Horizontal, Axis 2 (Y) - Vertical |
|Right Stick |N/A |Axis 3 - Horizontal, Axis 4 - Vertical |
|Dpad Up|joystick button 4 |Basic profile only: Axis 2 (Y) |
|Dpad Right |joystick button 5 |Basic profile only: Axis 1 (X) |
|Dpad Down |joystick button 6 |Basic profile only: Axis 2 (Y) |
|Dpad Left |joystick button 7|Basic profile only: Axis 1 (X) |
|Pause |joystick button 0 |N/A |
|L1/R1 |joystick button 8 / joystick button 9 |joystick axis 8 / joystick axis 9 |
|L2/R2 |joystick button 10 / joystick button 11 |joystick axis 10 / joystick axis 11 |

##Final notes on Game Controller API support
Unity only includes the Game Controller framework in the project if a script in the project references `Input.GetJoystickNames`. Unity iOS Runtime loads the framework dynamically, if it is available. For older iOS versions it returns an empty list of controllers.

Apple documentation explicitly states that controller input must be optional and your game should be playable without them.
