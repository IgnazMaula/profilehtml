<head>
    ....
    <link href="_content/Syncfusion.Blazor.Themes/bootstrap5.css" rel="stylesheet" />
    <script src="_content/Syncfusion.Blazor.Core/scripts/syncfusion-blazor.min.js" type="text/javascript"></script>
</head>

@page "/"

<PageTitle>Index</PageTitle>

<h1>Hello, world!</h1>

Welcome to your new app.

<SfDiagramComponent Height="600px" Nodes="@nodes" Connectors="@connectors" NodeCreating="@OnNodeCreating" ConnectorCreating="@OnConnectorCreating" Constraints="@DiagramConstraints">
    <DataSourceSettings ID="Id" ParentID="Parent" DataSource="Locations"> </DataSourceSettings>
    <Layout Type="LayoutType.HierarchicalTree" @bind-HorizontalSpacing="@HorizontalSpacing" @bind-VerticalSpacing="@VerticalSpacing">
    </Layout>
    <SnapSettings>
        <HorizontalGridLines LineColor="white" LineDashArray="2,2">
        </HorizontalGridLines>
        <VerticalGridLines LineColor="white" LineDashArray="2,2">
        </VerticalGridLines>
    </SnapSettings>
</SfDiagramComponent>

@code
{
    DiagramConstraints DiagramConstraints = DiagramConstraints.None;

    List<Location> Locations = new List<Location>() {
        new Location() {Id="1", Name="L1"},
        new Location() {Id="2", Name="L2", Parent="1"},
        new Location() {Id="3", Name="L3", Parent="1"},
        new Location() {Id="4", Name="L4", Parent="2"},
        new Location() {Id="5", Name="L5", Parent="4"},
        new Location() {Id="6", Name="L6", Parent="4"},
        new Location() {Id="7", Name="L7", Parent="1"},
        new Location() {Id="8", Name="L8"},
    };

    int left = 40;
    int top = 50;
    DiagramObjectCollection<Node> nodes = new DiagramObjectCollection<Node>();
    DiagramObjectCollection<Connector> connectors = new DiagramObjectCollection<Connector>();
    int HorizontalSpacing = 40;
    int VerticalSpacing = 40;

    private void OnNodeCreating(IDiagramObject obj)
    {
        Node node = obj as Node;
        node.Height = 100;
        node.Width = 200;
        var x = node.ID;
        //Initializing the default node's shape style.
        node.Style = new ShapeStyle() { Fill = "darkcyan", StrokeWidth = 3, StrokeColor = "Black" };
        if (node.Data is Location location)
        {
            // Set the Content property of the ShapeAnnotation
            node.Annotations = new DiagramObjectCollection<ShapeAnnotation>()
            {
                new ShapeAnnotation
                {
                    Content = location.Name,
                    Style = new TextStyle() { Color = "white", Bold = true },
                }
            };
        }
    }

    private void OnConnectorCreating(IDiagramObject connector)
    {
        (connector as Connector).Type = ConnectorSegmentType.Orthogonal;
    }

    public class Location
    {
        public string Id { get; set; }
        public string Name { get; set; }
        public string Parent { get; set; }
    }
}
