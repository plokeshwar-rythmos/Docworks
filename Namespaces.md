Namespaces
==========


As projects become larger and the number of scripts increases, the likelihood of having clashes between script class names grows ever greater. This is especially true when several programmers are working on different aspects of the game separately and will eventually combine their efforts in one project. For example, one programmer may be writing the code to control the main player character while another writes the equivalent code for the enemy. Both programmers may choose to call their main script class _Controller_, but this will cause a clash when their projects are combined.

To some extent, this problem can be avoided by adopting a naming convention or by renaming classes whenever a clash is discovered (eg, the classes above could be given names like _PlayerController_ and _EnemyController_). However, this is troublesome when there are several classes with clashing names or when variables are declared using those names - each mention of the old class name must be replaced for the code to compile.

The C# language offers a feature called **namespaces** that solves this problem in a robust way. A namespace is simply a collection of classes that are referred to using a chosen prefix on the class name. In the example below, the classes _Controller1_ and _Controller2_ are members of a namespace called _Enemy_:



````
namespace Enemy {
	public class Controller1 : MonoBehaviour {
		...
	}
	
	public class Controller2 : MonoBehaviour {
		...
	}
}

````

In code, these classes are referred to as `Enemy.Controller1` and `Enemy.Controller2`, respectively. This is better than renaming the classes insofar as the namespace declaration can be bracketed around existing class declarations (ie, it is not necessary to change the names of all the classes individually). Furthermore, you can use multiple bracketed namespace sections around classes wherever they occur, even if those classes are in different source files.

You can avoid having to type the namespace prefix repeatedly by adding a **using** directive at the top of the file.


	 using Enemy;

This line indicates that where the class names _Controller1_ and _Controller2_ are found, they should be taken to mean _Enemy.Controller1_ and _Enemy.Controller2_, respectively. If the script also needs to refer to classes with the same name from a different namespace (one called _Player_, say), then the prefix can still be used. If two namespaces that contain clashing class names are imported with using directives at the same time, the compiler will report an error.
