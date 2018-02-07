#Spatial Mapping basic low level API usage
<!-- https://trello.com/c/Qw7imxOL -->
The low level API provided for __Spatial Mapping__ offers very fine-grained control over a complicated system. 

The common usage patterns listed below may be useful:

##SurfaceObserver
The most important member of __Spatial Mapping__ low level API is the [SurfaceObserver](ScriptRef:XR.WSA.SurfaceObserver.html). This offers an insight into the device's understanding of the real world.

A __SurfaceObserver__ describes a volume in the world and reports which __Surfaces__ have been added, updated, or removed. Applications can then asynchronously request Mesh data with or without physics collision data. When the request has been filled, another callback informs the application that the data is ready.

__SurfaceObserver__ provides the following basic functionality:

1. It issues callbacks on-demand for __Surface__ changes like additions, removals, and updates.
1. It provides an interface for requesting Mesh data corresponding to a known __Surface__.
1. It issues callbacks when the requested Mesh data is available for use.
1. It provides ways of defining the location and volume of the __SurfaceObserver__.

##SurfaceData
The other significant moving part is the [SurfaceData](ScriptRef:XR.WSA.SurfaceData.html) object. This contains all the information required to build and report on a __Surface's__ Mesh data. 

A filled-out __SurfaceData__ is passed to the system in a [RequestMeshAsync](ScriptRef:XR.WSA.SurfaceObserver.RequestMeshAsync.html) call. When the Mesh data is ready, a matching __SurfaceData__ is returned in the "data ready" callback supplied at request time. This allows the application to determine precisely which __Surface__ the data corresponds to, with no ambiguity.

You should fill out the __SurfaceData__ based on the information your application needs. This includes passing in:

* [WorldAnchor](ScriptRef:XR.WSA.WorldAnchor.html)
* [MeshFilter](ScriptRef:MeshFilter.html)
* [MeshCollider](ScriptRef:MeshCollider.html) (if physics data is required)
* the triangles per cubic meter of the generated Mesh that you want
* the Surface ID 

The system throws argument exceptions when calling `RequestMeshAsync` with an incorrectly configured __SurfaceData__. Note that even if a `RequestMeshAsync` does not throw argument exceptions, there is no guarantee that Mesh data is created and returned.

##A simple use case
This sample outlines a basic use of this API.

````
using UnityEngine;
using UnityEngine.VR;
using UnityEngine.VR.WSA;
using UnityEngine.Rendering;
using UnityEngine.Assertions;
using System;
using System.Collections;
using System.Collections.Generic;

public enum BakedState {
	NeverBaked = 0,
	Baked = 1,
	UpdatePostBake = 2
}

// Data that is kept to prioritize surface baking.
class SurfaceEntry {
	public GameObject  m_Surface; // the GameObject corresponding to this surface
	public int         m_Id; // ID for this surface
	public DateTime    m_UpdateTime; // update time as reported by the system
	public BakedState  m_BakedState;
	public const float c_Extents = 5.0f;
}

public class SMSample : MonoBehaviour {
	// This observer is the window into the spatial mapping world.  
	SurfaceObserver	m_Observer;

	// This dictionary contains the set of known spatial mapping surfaces.
	// Surfaces are updated, added, and removed on a regular basis.
	Dictionary<int, SurfaceEntry> m_Surfaces;

	// This is the material with which the baked surfaces are drawn.  
	public Material m_drawMat;

	// This flag is used to postpone requests if a bake is in progress.  Baking mesh
	// data can take multiple frames.  This sample prioritizes baking request
	// order based on surface data surfaces and will only issue a new request
	// if there are no requests being processed.
	bool m_WaitingForBake;

	// This is the last time the SurfaceObserver was updated.  It is updated no 
	// more than every two seconds because doing so is potentially time-consuming.
	float m_lastUpdateTime;

	void Start () {
		m_Observer = new SurfaceObserver ();
		m_Observer.SetVolumeAsAxisAlignedBox (new Vector3(0.0f, 0.0f, 0.0f), 
			new Vector3 (SurfaceEntry.c_Extents, SurfaceEntry.c_Extents, SurfaceEntry.c_Extents));
		m_Surfaces = new Dictionary<int, SurfaceEntry> ();
		m_WaitingForBake = false;
		m_lastUpdateTime = 0.0f;
	}
	
	void Update () {
		// Avoid calling Update on a SurfaceObserver too frequently.
		if (m_lastUpdateTime + 2.0f < Time.realtimeSinceStartup) {
			// This block makes the observation volume follow the camera.
			Vector3 extents;
			extents.x = SurfaceEntry.c_Extents;
			extents.y = SurfaceEntry.c_Extents;
			extents.z = SurfaceEntry.c_Extents;
			m_Observer.SetVolumeAsAxisAlignedBox (Camera.main.transform.position, extents);

			try {
				m_Observer.Update (SurfaceChangedHandler);
			} catch {
				// Update can throw an exception if the specified callback was bad.
				Debug.Log ("Observer update failed unexpectedly!");
			}

			m_lastUpdateTime = Time.realtimeSinceStartup;
		}


		if (!m_WaitingForBake) {
			// Prioritize older adds over other adds over updates.
			SurfaceEntry bestSurface = null;
			foreach (KeyValuePair<int, SurfaceEntry> surface in m_Surfaces) {
				if (surface.Value.m_BakedState != BakedState.Baked) {
					if (bestSurface == null) {
						bestSurface = surface.Value;
					} else {
						if (surface.Value.m_BakedState < bestSurface.m_BakedState) {
							bestSurface = surface.Value;
						} else if (surface.Value.m_UpdateTime < bestSurface.m_UpdateTime) {
							bestSurface = surface.Value;
						}
					}
				}
			}
			if (bestSurface != null) {
				// Fill out and dispatch the request.
				SurfaceData sd;
				sd.id.handle = bestSurface.m_Id;
				sd.outputMesh = bestSurface.m_Surface.GetComponent<MeshFilter> ();
				sd.outputAnchor = bestSurface.m_Surface.GetComponent<WorldAnchor> ();
				sd.outputCollider = bestSurface.m_Surface.GetComponent<MeshCollider> ();
				sd.trianglesPerCubicMeter = 300.0f;
				sd.bakeCollider = true;
				try {
					if (m_Observer.RequestMeshAsync(sd, SurfaceDataReadyHandler)) {
						m_WaitingForBake = true;
					} else {
						// A return value of false when requesting meshes 
						// typically indicates that the specified surface
						// ID specified was invalid.
						Debug.Log(System.String.Format ("Bake request for {0} failed.  Is {0} a valid Surface ID?", bestSurface.m_Id));
					}
				}
				catch {
					// Requests can fail if the data struct is not filled out
					// properly.  
					Debug.Log (System.String.Format("Bake for id {0} failed unexpectedly!", bestSurface.m_Id));
				}
			}
		}
	}

	// This handler receives surface changed events and is propagated by the 
	// Update method on SurfaceObserver.  
	void SurfaceChangedHandler (SurfaceId id, SurfaceChange changeType, Bounds bounds, DateTime updateTime) {
		SurfaceEntry entry;
		switch (changeType) {
			case SurfaceChange.Added:
			case SurfaceChange.Updated:
			if (m_Surfaces.TryGetValue(id.handle, out entry)) {
				// If this surface has already been baked, mark it as needing bake
				// in addition to the update time so the "next surface to bake" 
				// logic will order it correctly.  
				if (entry.m_BakedState == BakedState.Baked) {
					entry.m_BakedState = BakedState.UpdatePostBake;
					entry.m_UpdateTime = updateTime;
				}
			} else {
				// This is a brand new surface so create an entry for it.
				entry = new SurfaceEntry ();
				entry.m_BakedState = BakedState.NeverBaked;
				entry.m_UpdateTime = updateTime;
				entry.m_Id = id.handle;
				entry.m_Surface = new GameObject (System.String.Format("Surface-{0}", id.handle));
				entry.m_Surface.AddComponent<MeshFilter> ();
				entry.m_Surface.AddComponent<MeshCollider> ();
				MeshRenderer mr = entry.m_Surface.AddComponent<MeshRenderer> ();
				mr.shadowCastingMode = ShadowCastingMode.Off;
				mr.receiveShadows = false;
				entry.m_Surface.AddComponent<WorldAnchor> ();
				entry.m_Surface.GetComponent<MeshRenderer> ().sharedMaterial = m_drawMat;
				m_Surfaces[id.handle] = entry;
			}
			break;

			case SurfaceChange.Removed:
			if (m_Surfaces.TryGetValue(id.handle, out entry)) {
				m_Surfaces.Remove (id.handle);
				Mesh mesh = entry.m_Surface.GetComponent<MeshFilter> ().mesh;
				if (mesh) {
					Destroy (mesh);
				}
				Destroy (entry.m_Surface);
			}
			break;
		}
	}

	void SurfaceDataReadyHandler(SurfaceData sd, bool outputWritten, float elapsedBakeTimeSeconds) {
		m_WaitingForBake = false;
		SurfaceEntry entry;
		if (m_Surfaces.TryGetValue(sd.id.handle, out entry)) {
			// These two asserts are checking that the returned filter and WorldAnchor
			// are the same ones that the data was requested with.  That should always
			// be true here unless code has been changed to replace or destroy them.
			Assert.IsTrue (sd.outputMesh == entry.m_Surface.GetComponent<MeshFilter>());
			Assert.IsTrue (sd.outputAnchor == entry.m_Surface.GetComponent<WorldAnchor>());
			entry.m_BakedState = BakedState.Baked;
		} else {
			Debug.Log (System.String.Format("Paranoia:  Couldn't find surface {0} after a bake!", sd.id.handle));
			Assert.IsTrue (false);
		}
	}
}
````

Note that call [Update](ScriptRef:XR.WSA.SurfaceObserver.Update.html) can be resource-intensive and should not be called more often than an application requires. Calling __Update__ once every three seconds should be sufficient for most use cases.
