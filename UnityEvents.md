UnityEvents
=================

UnityEvents are a way of allowing user driven callback to be persisted from edit time to run time without the need for additional programming and script configuration.

UnityEvents are useful for a number of things:

- Content driven callbacks

- Decoupling systems

- Persistent callbacks

- Preconfigured call events



`UnityEvent`s can be added to any `MonoBehaviour` and are executed from code like a standard .net delegate. When a `UnityEvent` is added to a `MonoBehaviour` it appears in the Inspector and persistent callbacks can be added.


`UnityEvent`s have similar limitations to standard delegates. That is, they hold references to the element that is the target and this stops the target being garbage collected. If you have a UnityEngine.Object as the target and the native representation disappears the callback will not be invoked. 


##Using UnityEvents


To configure a callback in the editor there are a few steps to take:

0. Make sure your script imports/uses `UnityEngine.Events`.

1. Select the + icon to add a slot for a callback

2. Select the UnityEngine.Object you wish to receive the callback (You can use the object selector for this)

3. Select the function you wish to be called

4. You can add more then one callback for the event



When configuring a `UnityEvent` in the Inspector there are two types of function calls that are supported:


- Static. Static calls are preconfigured calls, with preconfigured values that are set in the UI. This means that when the callback is invoked, the target function is invoked with the argument that has been entered into the UI.
- Dynamic. Dynamic calls are invoked using an argument that is sent from code, and this is bound to the type of UnityEvent that is being invoked. The UI filters the callbacks and only shows the dynamic calls that are valid for the UnityEvent.



##Generic UnityEvents


By default a `UnityEvent` in a `Monobehaviour` binds dynamically to a void function. This does not have to be the case as dynamic invocation of UnityEvents supports binding to functions with up to 4 arguments. To do this you need to define a custom `UnityEvent` class that supports multiple arguments. This is quite easy to do:


    [Serializable]

    public class StringEvent : UnityEvent <string> {}



By adding an instance of this to your class instead of the base `UnityEvent` it will allow the callback to bind dynamically to string functions. 



This can then be invoked by calling the `Invoke()` function with a `string` as argument.



UnityEvents can be defined with up to 4 arguments in their generic definition.

