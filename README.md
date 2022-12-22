# Owl and Jackalope UX

![GitHub release (latest by date)](https://img.shields.io/github/v/release/AnonyBob/OwlAndJackalope.UX) 
[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

Owl and Jackalope UX is a simple data binding and UX prototyping tool designed to make iterating on UI screens without needing to write code to back up the original models. It also includes several quality of life features intended to make reusing data models and UX states easier.

## Features
1. Create references of data models to use during prototyping without needing to load rest of game.
2. Display live data in editor for easy debugging and allow for manipulation to test responsiveness of scenes.
3. Establish two way data binding with Unity GUI elements.
4. Setup conditional checks with multiple elements to control the state of the UI screens.
5. Inject actual data models at runtime based on your personal project needs.

## Installation
Owl and Jackalope UX can be installed via the Unity package manager by following these [instructions](https://docs.unity3d.com/Manual/upm-ui-giturl.html) with the git url of:
https://github.com/AnonyBob/OwlAndJackalope.UX.git?path=/Assets/OwlAndJackalope.UX

or by adding

```"com.owlandjackalope.ux": "https://github.com/AnonyBob/OwlAndJackalope.UX.git?path=/Assets/OwlAndJackalope.UX"```

to the Packages/manifest.json file

or by downloading the released Unity Package.

## Usage

### References
References form the backbone of this plugin. A ```IReference``` is simply a collection of ```IDetails```. Each ```IDetail``` is a singular piece of data. Each ```IReference``` can have an arbitrary number of ```IDetails``` each retrievable by a unique string name. When the contents of a ```IDetail``` change it will send out a C# event that can be used to trigger updates in other ```IDetails``` or within the Scene. To create new References you can implement the ```IReference``` interface and expose ```IDetails``` in a manner that fits the need of your project, you can use the ```BaseReference``` which will maintain the ```IDetails``` in a simple Dictionary, or you can simply define them within the Unity Editor by using the Reference Module or Reference Templates. ```IReferences``` were designed to represent models that are assembled using composition and can be thought of as simple key value stores.

#### Reference Module
```ReferenceModules``` are the in game access points for ```IReferences``` and should be your starting point when creating new user experiences. ```ReferenceModules``` allow you to define a Reference that can be referenced in the editor while setting up binders in the Scene. You can create ```IDetails``` with a unique name and value. It supports most primitive types and a couple of commonly used Unity types like Gameobjects, Vector3, and Color as well as Lists of those types. When you enter playmode, these values will now update to reflect their current values and can be updated through the editor to drive further changes in the UI.

#### Adding New Detail Types
You can add support for new detail types in the inspector by creating serialized instances of them. Simply create a new type that extends the ```AbstractSerializedDetail``` class. The inspector will find all instances in code when determine which elements should be available. There are some subclasses that provide the majority of the functionality already. Such as the ```SerializedValueDetail``` and ```SerializedListValueDetail```. Additionally, you can use the ```SerializedDetailDisplay``` attribute to define what name should appear in the drop down and if it should go within a subfolder. These assume that the details will exist on a single row. If you can fit it on a single row in the inspector then there is no need to create a new property drawer. However, if you need to handle some custom drawing you can override aspects of the ```SerializedValueDetailPropertyDrawer``` or create your own.

#### Detail Binders

```DetailBinders``` are the simple components that can be attached to various elements in the UI that respond to changes in a specified detail and update the resulting visuals using some custom code per Binder definition. A few commonly used binders such as the ```TextDetailBinder``` have already been added that allow you to construct strings out of a collection of details like shown above. Some things may require the creation of a new ```DetailBinders``` in order for you to properly bind changes the UI.

New Binders can be created by extending the ```AbstractDetailBinder```. Each Binder can have multiple ```Observers```. ```Observers``` are wrappers that point to the ```IDetails``` exposed in the ```ReferenceModule```. This wrapper maintains access to the correct ```IDetail``` even if the the Reference in the ```ReferenceModule``` were to change via code in some way. Each ```Observer``` can be typed or untyped. By adding the ```DetailType``` attribute you can define which types of ```IDetails``` you would like to display in the Editor dropdown. Below you can see an example of how one might bind to multiple different types. In this case, all numeric types supported by the ```ReferenceModule```. For Lists of Details you can use the ```ListObserver``` instead. It will provide additional methods for working with lists like adding and removing.

Each ```Observer``` must be initialized. This should most likely happen in the Start method of the Binder. It can be initialized with or without a Handler method. The Handler method will run everytime there is a change to the observed detail. The Handler method will run the first time during initialization unless the suppressInitial boolean is set to true. The Handler is where the code that responds to the data changes will run. In the below example, whenever the color changes we will set the color on the graphic to the new color defined in the observed ```IDetail```. Observers are bidirectional objects. So if the Detail is considered mutable then you can use the Observers to make changes to the Detail in the reference as well. Attempting to do so on non-mutable details will throw an exception. The Observer has a boolean value where you can check the mutability of the observed detail.

***AbstractDetailBinder handles IDetail name changes. If you do opt to create your Binder from scratch it is important that you ensure that name changes are also handled by the your new binder type.***

#### Details Providers
```DetailsProviders``` are components that provide a collection of Details to an existing ```ReferenceModule``` at start up. Up to this point, you have been working with a Reference that only exists in the ```ReferenceModule```, but in a real game you will want your model to get backed by actual data. You can create a new component that extends the ```DetailsProvider``` class in order to do this. At start up the ```ReferenceModule``` will grab this component and request the Reference from the provider. This Reference can be retrieved in whatever way makes sense for your game. The provided Reference will override any ```IDetails``` contained in the ```ReferenceModule's``` default Reference with its own. Effectively updating any ```DetailBinders``` to point to the new Reference. However, if there are any ```IDetails``` that exist in the ```ReferenceModule``` and not in the new Reference, then they will continue to exist as well. In this way, you can combine view only ```IDetails``` with backing models and access them from the same source instead of maintaining two sets of models or cramming view and data only stuff into a single object.

```ReferenceModules``` can have their Reference set via code by using the exposed Reference property at anytime during gameplay. ```DetailsProviders``` simply provide a mechanism to assign the Reference at start up without having to access the ```ReferenceModule``` externally. A single example ```TemplateReferenceProvider``` has been added for testing purposes. It will provide the Reference contained in the assigned ```ReferenceTemplate``` and has a context menu item to set the Reference value for testing changing the Reference at different times during execution.

#### Reference Templates
```ReferenceTemplates``` are Scriptable Objects that contain a Reference and behave almost identically to ```ReferenceModules```. These can be used to assign ```IDetails``` to ```ReferenceModules``` and as example models to stub into your game for testing. ```ReferenceModules``` have an Import button that can be used to import the ```IDetails``` from an existing ```ReferenceTemplate```. This will only add ```IDetails``` that didn't originally existing the ```ReferenceModule```. All other ```IDetails``` will be preserved. ```ReferenceModules``` can now export ```ReferenceTemplates``` via a button in the inspector.

### Conditions and Condition Binders

