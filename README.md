@using Syncfusion.Blazor.Diagram

<SfDiagramComponent Height="600px" Nodes="@NodeCollection" Connectors="@connectors" NodeCreating="@OnNodeCreating" ConnectorCreating="@OnConnectorCreating">
    <Layout Type="LayoutType.OrganizationalChart" @bind-HorizontalSpacing="@HorizontalSpacing" @bind-VerticalSpacing="@VerticalSpacing">
    </Layout>
</SfDiagramComponent>

@code 
{
    //Defines diagram's connector collection.
    DiagramObjectCollection<Connector> connectors = new DiagramObjectCollection<Connector>();
    //Defines diagram's node collection.
    DiagramObjectCollection<Node> NodeCollection = new DiagramObjectCollection<Node>();

    int HorizontalSpacing = 40;
    int VerticalSpacing = 40;

    private void OnNodeCreating(IDiagramObject obj)
    {
        Node node = obj as Node;
        node.Height = 40;
        node.Width = 100;
        //Initializing the default node's shape style.
        node.Style = new ShapeStyle() { Fill = "darkcyan", StrokeWidth = 3, StrokeColor = "Black" };
        node.Annotations = new DiagramObjectCollection<ShapeAnnotation>()
        {
            new ShapeAnnotation { Style = new TextStyle() { Color = "white", Bold = true },Content = node.Annotations[0].Content }
        };
    }

    private void OnConnectorCreating(IDiagramObject connector)
    {
        (connector as Connector).Type = ConnectorSegmentType.Orthogonal;
    }

    protected override void OnInitialized()
    {
        NodeCollection = new DiagramObjectCollection<Node>()
        {
            new Node() { ID = "node1",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Project Management" } } },
            new Node() { ID = "node2",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "R&D Team" } } },
            new Node() { ID = "node3",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Philosophy" } } },
            new Node() { ID = "node4",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Organization" } } },
            new Node() { ID = "node5",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Technology" } } },
            new Node() { ID = "node6",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Funding" } } },
            new Node() { ID = "node7",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Resource-Allocation" } } },
            new Node() { ID = "node8",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Targeting" } } },
            new Node() { ID = "node9",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Evaluation" } } },
            new Node() { ID = "node10",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "HR-Team" } } },
            new Node() { ID = "node11",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Recruitment" } } },
            new Node() { ID = "node12",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Employee-Relation" } } },
            new Node() { ID = "node13",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Production & Sales Team" } } },
            new Node() { ID = "node14",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Design" } } },
            new Node() { ID = "node15",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Operation" } } },
            new Node() { ID = "node16",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Support" } } },
            new Node() { ID = "node17",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Quality Assurance" } } },
            new Node() { ID = "node18",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Customer Interaction" } } },
            new Node() { ID = "node19",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Support and Maintenance" } } },
            new Node() { ID = "node20",Annotations = new DiagramObjectCollection<ShapeAnnotation>() { new ShapeAnnotation { Content = "Task Coordination" } } }
        };        
        connectors = new DiagramObjectCollection<Connector>()
        {
            new Connector() { ID = "connector1", SourceID = "node1", TargetID = "node2" },
            new Connector() { ID = "connector2", SourceID = "node1", TargetID = "node10" },
            new Connector() { ID = "connector3", SourceID = "node1", TargetID = "node13" },
            new Connector() { ID = "connector4", SourceID = "node2", TargetID = "node3" },
            new Connector() { ID = "connector5", SourceID = "node2", TargetID = "node4" },
            new Connector() { ID = "connector6", SourceID = "node2", TargetID = "node5" },
            new Connector() { ID = "connector7", SourceID = "node2", TargetID = "node6" },
            new Connector() { ID = "connector8", SourceID = "node2", TargetID = "node7" },
            new Connector() { ID = "connector9", SourceID = "node2", TargetID = "node8" },
            new Connector() { ID = "connector10", SourceID = "node2", TargetID = "node9" },
            new Connector() { ID = "connector11", SourceID = "node2", TargetID = "node11" },
            new Connector() { ID = "connector12", SourceID = "node10", TargetID = "node12" },
            new Connector() { ID = "connector13", SourceID = "node10", TargetID = "node14" },
            new Connector() { ID = "connector14", SourceID = "node13", TargetID = "node15" },
            new Connector() { ID = "connector15", SourceID = "node13", TargetID = "node16" },
            new Connector() { ID = "connector16", SourceID = "node13", TargetID = "node17" },
            new Connector() { ID = "connector17", SourceID = "node13", TargetID = "node18" },
            new Connector() { ID = "connector18", SourceID = "node13", TargetID = "node19" },
            new Connector() { ID = "connector19", SourceID = "node13", TargetID = "node20" }
        };
    }
}
