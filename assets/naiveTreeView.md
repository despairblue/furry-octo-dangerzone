sequenceDiagram
  participant Scene Graph
  participant Tree View
  participant Tree Node
  Scene Graph->> Tree View: traverse and create Tree View
  loop main
    opt add node to tree view
      Tree View->> Tree View: add new node
      Tree View->> Scene Graph: add new node
    end
    opt remove node
      Tree View->> Tree View: remove node
      Tree View->> Scene Graph: remove node
    end
    opt attribute mutation in Tree View
      Tree Node->> Scene Graph: change attribute
    end
    opt attribute mutation in Scene Graph (3D canvas)
      Scene Graph--x Tree Node: notify
      Tree Node->> Tree Node: update attributes
    end
  end
