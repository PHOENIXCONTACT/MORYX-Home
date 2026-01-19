# Component Design Guidelines

## To be or not to be a component

There are way too many similar definitions of a software component. For a comprehensive tutorial on structuring components please see [NDepend White Book](http://www.ndepend.com/Res/NDependWhiteBook_Namespace.pdf). For this component design guideline, components will be considered that are used by a single non-trivial PluginService.

## Component definition

A component in the scope of this guideline is an **aggregate of classes and associated types**. A class is mostly too lightweight to define a component. Therefore a component will almost always include several classes or interfaces and their associated types (enumerations, structures, exceptions, attributes, delegates…). All these types are grouped by a common well-understood semantic concept that gives its name to the component.

## Module=Lifecycle, Component=Work

The ModuleController is responsible for the life-cycle of the plugin. As such, it will create, initialize, start, stop and dispose the components needed for doing the actual work.

## Sub-Namespace=Sub-Feature

Large components should be divided into sub-namespaces. A sub-namespaces must correspond to a sub-feature of the component. Sub-namespaces should not know each other. This is done to reduce any chance of coupling. The parent component is the PluginService and plays the role of a mediator. The mediator is responsible for making sub-components communicate. The preferred way of achieving this is comprehensive usage of an IoC mechanism.

## Namespace conventions

Namespaces should be used to logically group components. As a summary, the typical namespaces of a component (be it a ServerModule or not) is given below. Namespace dependencies should be avoided.

| Namespace            | Depends on | Used/referenced by               | Description                                                                                           |
|----------------------|------------|----------------------------------|-------------------------------------------------------------------------------------------------------|
| Moryx.ABC            | none       | Plugins <br> Other ServerModules | Root namespace of component ABC, only the component public surface API                                |
| Moryx.ABC.ModuleName | Moryx.ABC  | none                             | Main component, implementation of public surface API, definition of ServerModule and internal Plugins |
| Moryx.ABC.Endpoints  | Moryx.ABC  | none                             | Exposes a Module's Facade externally.                                                                 |
| Moryx.ABC.ModulePluginXY | Moryx.ABC <br> Moryx.ABC.ModuleName | none | A module plugin XY, a specific and cohesive set of features                                          |