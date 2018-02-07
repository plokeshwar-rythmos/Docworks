Layers
======


__Layers__ are most commonly used by __Cameras__ to render only a part of the scene, and by __Lights__ to illuminate only parts of the scene. But they can also be used by raycasting to selectively ignore colliders or to create [collisions](LayerBasedCollision).


Creating Layers
---------------


The first step is to create a new layer, which we can then assign to a __GameObject__. To create a new layer, open the Edit menu and select __Project Settings-&gt;Tags and Layers__.

We create a new layer in one of the empty User Layers. We choose layer 8.


![](../uploads/Main/Layer-CreateNewLayer.png) 


Assigning Layers
----------------


Now that you have created a new layer, you have to assign the layer to one of the game objects.


![](../uploads/Main/Layer-ChooseLayer.png) 

In the tag manager we assigned the Player layer to be in layer 8.


Drawing only a part of the scene with the camera's culling mask
---------------------------------------------------------------


Using the camera's culling mask, you can selectively render objects which are in one particular layer.
To do this, select the camera that should selectively render objects.

Modify the culling mask by checking or unchecking layers in the culling mask property.


![](../uploads/Main/Layer-CullingMask.png) 

Be aware that UI elements aren't culled. Screen space canvas children do not respect the camera's culling mask.


Casting Rays Selectively
------------------------


Using layers you can cast rays and ignore colliders in specific layers.
For example you might want to cast a ray only against the player layer and ignore all other colliders.

The Physics.Raycast function takes a bitmask, where each bit determines if a layer will be ignored or not.
If all bits in the layerMask are on, we will collide against all colliders.
If the layerMask = 0, we will never find any collisions with the ray.




````
// JavaScript example.

// bit shift the index of the layer to get a bit mask
var layerMask = 1 << 8;
// Does the ray intersect any objects which are in the player layer.
if (Physics.Raycast (transform.position, Vector3.forward, Mathf.Infinity, layerMask))
    print ("The ray hit the player");


// C# example.

int layerMask = 1 << 8;
		
// Does the ray intersect any objects which are in the player layer.
if (Physics.Raycast(transform.position, Vector3.forward, Mathf.Infinity, layerMask))
    Debug.Log("The ray hit the player");


````


In the real world you want to do the inverse of that however. We want to cast a ray against all colliders except those in the Player layer.



````
// JavaScript example.
function Update () {
  // Bit shift the index of the layer (8) to get a bit mask
  var layerMask = 1 << 8;
  // This would cast rays only against colliders in layer 8.
  // But instead we want to collide against everything except layer 8. The ~ operator does this, it inverts a bitmask.
  layerMask = ~layerMask;

  var hit : RaycastHit;
  // Does the ray intersect any objects excluding the player layer
  if (Physics.Raycast (transform.position, transform.TransformDirection (Vector3.forward), hit, Mathf.Infinity, layerMask)) {
    Debug.DrawRay (transform.position, transform.TransformDirection (Vector3.forward) * hit.distance, Color.yellow);
    print ("Did Hit");
  } else {
    Debug.DrawRay (transform.position, transform.TransformDirection (Vector3.forward) *1000, Color.white);
    print ("Did not Hit");
  }
}


// C# example.
void Update () {
    // Bit shift the index of the layer (8) to get a bit mask
    int layerMask = 1 << 8;
		
    // This would cast rays only against colliders in layer 8.
    // But instead we want to collide against everything except layer 8. The ~ operator does this, it inverts a bitmask.
    layerMask = ~layerMask;
	
    RaycastHit hit;
    // Does the ray intersect any objects excluding the player layer
    if (Physics.Raycast(transform.position, transform.TransformDirection (Vector3.forward), out hit, Mathf.Infinity, layerMask)) {
        Debug.DrawRay(transform.position, transform.TransformDirection (Vector3.forward) * hit.distance, Color.yellow);
        Debug.Log("Did Hit");
    } else {
        Debug.DrawRay(transform.position, transform.TransformDirection (Vector3.forward) *1000, Color.white);
        Debug.Log("Did not Hit");
    }
}

````

When you don't pass a layerMask to the Raycast function, it will only ignore colliders that use the IgnoreRaycast layer.
This is the easiest way to ignore some colliders when casting a ray.

__Note__: Layer 31 is used internally by the Editor's Preview window mechanics. To prevent clashes, do not use this layer.

---

* <span class="page-edit">2017-05-08  <!-- include IncludeTextAmendPageSomeEdit --></span>

* <span class="page-history">Culling mask information updated in Unity 2017.1</span> 
