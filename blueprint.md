![](http://g.gravizo.com/g?
digraph min{
    Balancer[label="Balancer\n[nginx]"];
    Server[label="Server\n[Java]"];
    Monitor[label="Monitor\n[Python]"];
    Mapper[label="Mapper\n[Python]"];
    WebConsole[label="WebConsole\n[nginx/Python/Django]"];
    GameStorage[label="data\n[Presistent]", shape="box"];
    MapStorage[label="map data\n[Presistent]", shape="box"];
    
    Monitor -> Server [label="TCP"];
    Balancer -> Monitor [label="TCP"];
    Balancer -> Server [label="TCP"];
    Balancer -> WebConsole [label="HTTP"];
    
    WebConsole -> Server [label="TCP"];
    WebConsole -> MapStorage [label="NFS"];
    
    Server -> GameStorage [label="NFS"];
    
    Mapper -> MapStorage [label="NFS"];
    Mapper -> GameStorage [label="NFS"];
    
}
)