sequenceDiagram
  participant MutationObserver
  participant Scene Graph
  participant Tree View
  Scene Graph->> Tree View: tell react to render Tree View
  loop observe Scene Graph
    opt add node
      Tree View->> Scene Graph: add new node
      Scene Graph--x MutationObserver: notify
    end
    opt remove node
      Tree View->> Scene Graph: remove node
      Scene Graph--x MutationObserver: notify
    end
    opt change attribute
      alt changed in Tree View
        Tree View->> Scene Graph: change attribute
        Scene Graph--x MutationObserver: notify
      else changed in Scene Graph (3D canvas)
        Scene Graph->> Scene Graph: mutate attribute by modifying the scene directly
        Scene Graph--x MutationObserver: notify
      end
    end
    MutationObserver->> Tree View: tell react to render Tree View
  end
