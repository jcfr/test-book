# Programming Annotations

## Python examples

### Add a Ruler via Python

Use this code to programatically add a ruler to the scene:
```
rulerNode = slicer.vtkMRMLAnnotationRulerNode()
rulerNode.SetPosition1(-10,-10,-10)
rulerNode.SetPosition2(10,10,10)
rulerNode.Initialize(slicer.mrmlScene)
```

### Access to Ruler Locations from Python

Starting from the ID of an Annotation hierarchy node that collects a group of rulers, you can get a ruler location using the following Python code:
```
# get the first list of annotations
listNodeID = "vtkMRMLAnnotationHierarchyNode2"
annotationHierarchyNode = slicer.mrmlScene.GetNodeByID(listNodeID)
# get the first in the list
listIndex = 0
annotation = annotationHierarchyNode.GetNthChildNode(listIndex).GetAssociatedNode()
coords1 = [0,0,0]
coords2 = [0,0,0]
annotation.GetPosition1(coords1)
annotation.GetPosition2(coords2)
print coords1, coords2
```

If you have the id of the ruler node, it's more direct:
```
rulerID = "vtkMRMLAnnotationRulerNode1"
ruler = slicer.mrmlScene.GetNodeByID(rulerID)
coords1 = [0,0,0]
coords2 = [0,0,0]
ruler.GetPosition1(coords1)
ruler.GetPosition2(coords2)
print coords1, coords2
```

### Placing the GUI into Ruler or ROI Place Mode via Python

To programatically set the mouse mode to placing rulers or ROIs use this code from Python:
```
selectionNode = slicer.mrmlScene.GetNodeByID("vtkMRMLSelectionNodeSingleton")
# place rulers
selectionNode.SetReferenceActivePlaceNodeClassName("vtkMRMLAnnotationRulerNode")
# to place ROIs use the class name vtkMRMLAnnotationROINode
interactionNode = slicer.mrmlScene.GetNodeByID("vtkMRMLInteractionNodeSingleton")
placeModePersistence = 1
interactionNode.SetPlaceModePersistence(placeModePersistence)
# mode 1 is Place, can also be accessed via slicer.vtkMRMLInteractionNode().Place
interactionNode.SetCurrentInteractionMode(1)
```

### Transforms

Individual annotations are transformable (able to be placed under a transform node), but lists are not. In order to apply a transform to a set of annotations, open the Python console (View -> Python Interactor) and use the following code. First you'll need to get the transform node MRML id from the Data module (click on Display MRML IDs), and save it to a variable:
```
transformNodeID = "vtkMRMLLinearTransformNode5"
```
Then get the id of the annotation hierarchy list node that holds the annotations that you wish to transform:
```
listNodeID = "vtkMRMLAnnotationHierarchyNode3"
```
Then use this code snippet to apply the transform to all the annotations under the hierarchy node:
```
annotationHierarchyNode = slicer.mrmlScene.GetNodeByID(listNodeID)
numNodes = annotationHierarchyNode.GetNumberOfChildrenNodes()
for i in range(numNodes):
  annotation = annotationHierarchyNode.GetNthChildNode(i).GetAssociatedNode()
  annotation.SetAndObserveTransformNodeID(transformNodeID)
```
Read more about transforming MRML nodes in [Transforms module documentation](../Transforms/README.md).

### Selection and interaction

This section indicates how to change in the GUI the mouse placing mode to start placing nodes.
 
For the selection and interaction nodes, make sure that you access the singleton nodes already in the scene (rather than making your own) via (caveat: this call works on displayable managers, you may have to go a different route to get at the application logic):
```
vtkMRMLApplicationLogic *mrmlAppLogic = this->GetMRMLApplicationLogic();
vtkMRMLInteractionNode *inode = mrmlAppLogic->GetInteractionNode();
vtkMRMLSelectionNode *snode = mrmlAppLogic->GetSelectionNode();
```

If you can't get at the mrml application logic, you can get the nodes from the scene:
```
vtkMRMLInteractionNode *interactionNode = vtkMRMLInteractionNode::SafeDownCast(this->GetMRMLScene()->GetNodeByID("vtkMRMLInteractionNodeSingleton"));
```
You can then call methods on the nodes or add your own event observers as in
```
vtkSlicerAnnotationModuleLogic::ObserveMRMLScene
```
You can see vtkSlicerAnnotationModuleLogic::ProcessMRMLNodesEvents to see how to respond to interaction node changes, but I don't think you'll need to do that.

Slicer4/Base/QTGUI/qSlicerMouseModeToolBar.cxx has a lot of the code you'll need as well, with a slightly different way of getting at the mrml app logic to access the nodes.

The calls you need to make to switch into placing rulers with the mouse are:
```
selectionNode->SetReferenceActivePlaceNodeClassName("vtkMRMLAnnotationRulerNode");
interactionNode->SetCurrentInteractionMode(vtkMRMLInteractionNode::Place);
```
If you don't set PlaceModePersistence on the interaction node, the mouse mode/current interaction mode will automatically go back to view transform after one ruler has been placed, and you just need to reset the current interaction mode for future placing (the active place node class name is persistent).

## More information

* [User manual](../../modules/annotations/README.md)
* [Source code]({{book.slicerSourceCodeBase}}/Modules/Loadable/Annotations)
