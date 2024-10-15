## Some concepts before designing the maps
- We have GeoHashing: everything is divided in to tiles
  ![image](https://github.com/user-attachments/assets/9022173b-ca03-41e4-8ab8-3cff977dd75a)
- As we need to only downlaod the relevant data for a path, we will download the details for the relevant tiles only.
- In the downloaded routing tiles, we will have road data, road data is like a graph, where intersections are like a node, and roads are like edges, then we need to apply dijstra or A* to calculate the path.
- We can have different types of routing tile for a region accprding to the detail. For examples, moving across cities might not provide me the street data instead only high ways.
  - street routing tile
  - main roads routing tile
  - highways routing tiles 
