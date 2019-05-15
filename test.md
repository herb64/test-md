# Vanishing static mesh on Static Mesh Components on recompile

This is a mini project to reproduce a problem in a completely reduced environment.

## Background
In my current project, I have the problem, that static meshes assigned to C++ Based StaticMeshComponents
disappear, as soon as the code gets recompiled, even if just a comment line is changed. In practice, this means, that after any
compile step, the components have to be readded/reworked.

I found many discussions on this. For example https://issues.unrealengine.com/issue/UE-63298 mentions this being
caused by changing a public property to private and also mentions not to hot reload when changing properties.

Indeed, I had UPROPERTIES in private sections (not changing them from public to private, they were 
private from the beginning), so I changed them to public. No success.

Avoiding the hot reload: I closed the UE4Editor, then compiled, reopened the editor. This should avoid any
hot reload in my understanding - but mesh is gone on reopen.

So I decided to make an absolute reduced mini project to reproduce this in a small project. These were the basic steps

* New C++ project, basic code
* Add BP_simplepawn blueprint class based on Pawn
* Create a blueprint spawnable C++ Class named SMC_test derived from StaticMeshComponent
* Add the SMC_test custom component to BP_simplepawn as root scene component
* Add a public UPROPERTY float value for testing (can be changed to see, if code was compiled and value appears in editor)
* Add the /Engine/BasicShapes/Cone mesh as Static Mesh property in the Blueprint Editor
* Place one instance of BP_simplepawn into the scene

## How to reproduce using this project

* Clone the project.
* Open in UE4 and check the blueprint: it comes with a SMC_test object set as default scene component with the Engine "Cone" static mesh
* Verify, that mesh is present
* Close the UE4 Editor to avoid hot reload at all
* In SMC_test.cpp: Change the value of the test variable from 5000 to another value. Keep the workaround code disabled.
* Compile from Visual Studio
* Reopen the UE4 Editor and check: the mesh will be gone and the new value of the variable should be updated, proving that the new compiled code has been used.

You may now play around, e.g. enabling the workaround or readding the mesh in editor.

## The "workaround" code

I created a very ugly workaround, implementing the constructor of the component to load the Cone static mesh manually. This can
be switched on/off using a define statement.
```
#ifdef WORKAROUND
	// We simply use the Engine Cone Basic Shape
	static ConstructorHelpers::FObjectFinder<UStaticMesh>SM_Cone(TEXT("/Engine/BasicShapes/Cone.Cone"));
	UStaticMesh* EngineConeMesh = SM_Cone.Object;
	if (EngineConeMesh != nullptr) {
		this->SetStaticMesh(EngineConeMesh);
	} else {
		UE_LOG(LogTemp, Error, TEXT("Could not set static mesh to component"));
	}
#endif
```
The workaround code is disabled after cloning the project.


## More 

BTW, sometimes, other issues did come up as well, e.g.

* Even on each load of the project, the static mesh is gone without compiler run and this can only be recovered by completely removing the component and adding it again.
* Running into ensure condition with the cone: failed to resolve archetype object - property values may be lost.

