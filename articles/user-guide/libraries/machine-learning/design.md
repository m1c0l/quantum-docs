---
author: cjgronlund
description: Learn the basic concepts of designing circuit models for the quantum circuit centric classifier.
ms.author: cgronlun
ms.date: 02/01/2021
ms.service: azure-quantum
ms.subservice: qsharp-guide
ms.topic: conceptual
no-loc: ['Q#', '$$v']
title: Design your own classifier with the QDK
uid: microsoft.quantum.libraries.overview.machine-learning.design
---

# Design your own classifier

In this guide you will learn the basic concepts behind the design of circuit models for the quantum circuit centric classifier.

As in classical deep learning, there is no general rule for choosing a specific architecture. Depending on the problem some architectures will perform better than others. But, there are some concepts that might be useful when designing the circuit:

- A large number of parameters implies a more flexible model, which may be suitable to draw complicated classification boundaries but which may also be more susceptible to overfitting.

- Entangling gates between qubits are essential to capture the correlations between the quantum features.

## How to build a classifier with Q\#

To build a classifier we are going to concatenate parametrized controlled rotations in our circuit model. To do it we can use the type [`ControlledRotation`](xref:Microsoft.Quantum.MachineLearning.ControlledRotation) defined in the Quantum Machine Learning library. This type accepts four arguments that determine: the index of the target qubit, the array of indices of the control qubits, the axis of rotation, and index of the associated parameter in the array of parameters defining the model.

Let's see an example of a classifier. In the [half-moons sample](https://github.com/microsoft/Quantum/tree/main/samples/machine-learning/half-moons), we can find the following classifier defined in the file `Training.qs`.

```qsharp
    function ClassifierStructure() : ControlledRotation[] {
        return [
            ControlledRotation((0, new Int[0]), PauliX, 4),
            ControlledRotation((0, new Int[0]), PauliZ, 5),
            ControlledRotation((1, new Int[0]), PauliX, 6),
            ControlledRotation((1, new Int[0]), PauliZ, 7),
            ControlledRotation((0, [1]), PauliX, 0),
            ControlledRotation((1, [0]), PauliX, 1),
            ControlledRotation((1, new Int[0]), PauliZ, 2),
            ControlledRotation((1, new Int[0]), PauliX, 3)
        ];
    }
 ```

What we are defining here is a function that returns an array of `ControlledRotation` elements, that together with an array of parameters and a bias will define our [`SequentialModel`](xref:Microsoft.Quantum.MachineLearning.SequentialModel). This type is fundamental in the Quantum Machine Learning library and defines the classifier. The circuit defined in the function above is part of a classifier in which each sample of the dataset contains two features. Therefore we only need two qubits. The graphical representation of the circuit is:

 ![Circuit model example](~/media/circuit_model_1.PNG)

Note that by default the operations of the Quantum Machine Learning library
measure the last qubit of the register to estimate the classification
probabilities. You should keep in mind this fact when designing your circuit.

## Use the library functions to write layers of gates

Suppose we have a dataset with 784 features per instance, for example, images of 28×28 pixels like the MNIST dataset. In this case, the width of the circuit becomes large enough so that writing by hand each individual gate becomes a possible but impractical task. This is why the Quantum Machine Learning library provides a set of tools to automatically generate layers of parametrized rotations. For instance, the function [`LocalRotationsLayer`](xref:Microsoft.Quantum.MachineLearning.LocalRotationsLayer) returns an array of uncontrolled single-qubit rotations along a given axis, with one rotation for each qubit in the register, each parametrized by a different model parameter. For example, `LocalRotationsLayer(4, X)` returns the following set of gates:

 ![Local rotations layer](~/media/local_rotations_layer.PNG)

We recommend you explore the [API reference of the Quantum Machine Learning library](xref:Microsoft.Quantum.MachineLearning) to discover all the tools available to streamline the circuit design.

## Next steps

 Try different structures in the examples provided by the samples. Do you see any changes in the performance of the model? In the next tutorial, [`Load your own datasets`](xref:microsoft.quantum.libraries.overview.machine-learning.load), you will learn how to load datasets to try and explore new architectures of classifiers.