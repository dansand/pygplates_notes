# Pygplates: features, objects and geoms

## Key points:

* Be careful when accessign the geometry of a resolved feature - you can end up with the unresolved(currecnt day coordinates) if use the wrong methodsd)
* When resolving you end up with lists of objects (all inherted from pygplates.ReconstructionGeometry ). The order of the objects in these lists is not fixed - it wil change weach type youy call resolve. 

# Features and geometries

## features

The feature is an abstract model of some geological or plate-tectonic object or concept of interest defined by the [GPlates Geological Information Model](http://www.gplates.org/docs/gpgim) (GPGIM)

Each feature has properties (the ones you would see if you clicked on  that feature in gplates) and a geometry (the point/line/polygon that  gplates displays). We can access these for each feature, e.g. print the
Plate ID.

Features have lots of attrubutes, and PyGplates uses getters and setters to modify these.

A [`feature`](https://www.gplates.org/docs/pygplates/generated/pygplates.Feature.html#pygplates.Feature) is essentially a list of [`properties`](https://www.gplates.org/docs/pygplates/generated/pygplates.Property.html#pygplates.Property) where each property has a [`name`](https://www.gplates.org/docs/pygplates/generated/pygplates.PropertyName.html#pygplates.PropertyName) and a [`value`](https://www.gplates.org/docs/pygplates/generated/pygplates.PropertyValue.html#pygplates.PropertyValue).



Features generally combine geometries and a number of other attributes, such as plate Id, valid times etc.  



## Geometries

There are four types of geometry:
`pygplates.PointOnSphere `  A point on the surface of the unit length sphere in 3D cartesian coordinates.

`pygplates.MultiPointOnSphere `A multi-point (collection of points) on the surface of the unit length sphere.

`pygplates.PolylineOnSphere` A polyline on the surface of the unit length sphere.
pygplates.PolygonOnSphere 	A  polygon on the surface of the unit length sphere.

All four above geometry types inherit from:
`pygplates.GeometryOnSphere` base class inherited by all derived classes representing geometries on the sphere. 

A polyline or a polygon is both a sequence of points and a sequence of segments (between adjacent points). Each segment is a great circle arc:

`pygplates.GreatCircleArc` 	A great-circle arc on the surface of the unit globe.

There is also a latitude/longitude version of a point:

`pygplates.LatLonPoint`Represents a point in 2D geographic coordinates (latitude and longitude).

```
print(typrint(type(ridge_feat.get_geometry()))
print(type(ridge_feat.get_geometry().get_segments()))
***
<class 'pygplates.PolylineOnSphere'>
<class 'pygplates.PolylineOnSphereArcsView'>
```

What is a Segment? 

* a segment is a pygplates.GreatCircleArc, That is - an implied GCA between two points. 

It seems like lots of features, like MORS are naturally segmented - into, i.e. ridges and transforms (although they are not tagged as such, and some individial transfrom or ridge section contain multiple segments)

## To numpy arrays

The base geometry class (`pygplates.GeometryOnSphere`) has the following methods

`get_points() `	          Returns a read-only sequence of points in this geometry.
`to_lat_lon_array() `       Returns the sequence of points as a numpy array of (latitude,longitude) pairs (in degrees).
`to_lat_lon_list()` 	Returns the sequence of points, in this geometry, as (latitude,longitude) tuples (in degrees).
`to_lat_lon_point_list()` Returns the sequence of points, in this geometry, as lat lon points.
`to_xyz_array() `	       Returns the sequence of points, in this geometry, as a numpy array of (x,y,z) triplets.
`to_xyz_list() `	         Returns the sequence of points, in this geometry, as (x,y,z) cartesian coordinate tuples.



```
polygon = feature.get_geometry()
points = polygon.get_points() #why this step?
print points.to_lat_lon_list()
```

## Coordinates systems



### Geocentric

### Local cartesian

A local cartesian coordinate system located at a point on the sphere.

- *magnitude* is the length of the 3D vector,
- *azimuth* is the angle (in radians) clockwise (East-wise) from North (from 0 to 2*PI),
- *inclination* is the angle (in radians) in the downward  direction (eg, PI/2 if vector aligned with Down axis, -PI/2 if aligned  with up direction and 0 if vector in tangent plane).

### NED



## Vectors



# Objects resulting from reconstruction and resolving

Here are the functions to reconstruct/resolve an object / plate boundary system to a time in the past:

```
pygplates.reconstruct(...) 	#not there's alos a reverse_reconstuct to go forward
pygplates.resolve_topologies(...)
```

One commonality is that the objects created by these two functions all inherit from a class called:

```
pygplates.ReconstructionGeometry 	
```

#### Classes resulting from reconstructing regular features at a particular reconstruction time:

`pygplates.ReconstructedFeatureGeometry` The geometry of a feature reconstructed to a geological time.
`pygplates.ReconstructedFlowline` The reconstructed history of plate motion away from a spreading ridge in the form of a path of points over geological time.
`pygplates.ReconstructedMotionPath` The reconstructed history of a plate’s motion in the form of a path of points over geological time.

#### Classes resulting from resolving topological features at a particular reconstruction time.
`pygplates.ResolvedTopologicalLine` 	The geometry of a topological line feature resolved to a geological time.
`pygplates.ResolvedTopologicalBoundary` 	The geometry of a topological boundary feature resolved to a geological time.
`pygplates.ResolvedTopologicalNetwork `	The geometry of a topological network feature resolved to a geological time.

#### A Further level of inhereted classes

The following class represents a sub-segment of a single resolved topological line, boundary or network.

`pygplates.ResolvedTopologicalSubSegment` The subset of vertices of a reconstructed topological section that contribute to the geometry of a resolved topology.

The following classes represent sub-segments shared by one or more resolved topological boundaries and/or networks.

`pygplates.ResolvedTopologicalSection` 	The sequence of shared sub-segments of a reconstructed topological section that uniquely contribute to the boundaries of one or more resolved topologies.

`pygplates.ResolvedTopologicalSharedSubSegment `	The shared subset of vertices of a reconstructed topological section that uniquely contribute to the boundaries of one or more resolved topologies.

When you run the followng code (a common starting poitnt in my analyis):

```java
resolved_topologies = []
shared_boundary_sections = []

pygplates.resolve_topologies(topology_features, rotation_model, 
                             resolved_topologies, time, 
                             shared_boundary_sections, anchor_plate_id)
```

The `resolved_ topologies` are type `pygplates.ResolvedTopologicalBoundary`

while shared_boundary_sections are type `pygplates.ResolvedTopologicalSection`

Let's see the steps involved in calculating a plate's area:

```python
area = resolved_topologies[0].get_resolved_geometry().get_area()
```

The important thing to  focus on here is the use of `get_resolved_geometry()`. 

As a rule of thumb, you always want to used the getters that refer to a resolved or reconstucted feature/geometry. See the following:

```python
sbt = shared_boundary_sections[0]
rtsss = sbt.get_shared_sub_segments()[0]
#the shared subsegments dude includes these methods, which one could easily confuse
rtsss.get_resolved_feature()
rtsss.get_feature()
```

```python
print(type(sbt))
print(type(rtsss))
...
<class 'pygplates.ResolvedTopologicalSection'>
<class 'pygplates.ResolvedTopologicalSharedSubSegment'>
```

Looking back at the dedinitiosn above, the `sbbt` object is insnanct of a class refers to the shared subset of vertices of a reconstructed topological section that uniquely contribute to the boundaries of one or more resolved topologies.

```python
print(rtsss.get_resolved_feature().get_geometry().get_centroid().to_lat_lon_array())
print(rtsss.get_geometry().get_centroid().to_lat_lon_array()[0])
print(rtsss.get_feature().get_geometry().get_centroid().to_lat_lon_array())
....
[[-14.86613096 176.59441637]]
[-14.86613096 176.59441637]
[[-22.14940785 179.97160827]]

```

This first two method get the reconstructed position,  ( I assume that  using the `get_feature` method gives the feature in it present day coordinates)

Interestingly, the `sbt` object does not have the `get_resolved_feature` method. Which seems very confusing to me. Instead you can use:

```python
print(sbt.get_topological_section_geometry().get_centroid().to_lat_lon_array())
print(sbt.get_topological_section().get_reconstructed_geometry().get_centroid().to_lat_lon_array())
print(sbt.get_feature().get_geometry().get_centroid().to_lat_lon_array())
...
[[-22.76331299 176.40671172]]
[[-22.76331299 176.40671172]]
[[-22.14940785 179.97160827]]

```



