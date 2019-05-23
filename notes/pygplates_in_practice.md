# PyGplates

## Absolute plate motions

A good paper on reference frames is Williams et al. Absolute plate motions since 130 Ma
constrained by subduction zone kinematics. 

They make the distinction between relative plate motion models RPM. which describes the relative motions of the plates (like most EarthByte stuff, they use Seton 2012), and the Absolute plate model (APM), which is basically just the motion of Africa in the Seton hierarchy. __All of the APMs used are available in the paper's online content.__

For times older than 83 Ma, the Pacific plate
is surrounded by subduction zones and cannot be linked to Africa
within an RPM model




## Rotations background

GPlates uses a plate-motion model representation in which plate motions are described in terms of relative rotations between pairs of plates. Thus, the full plate-motion model is a directed graph of rotations between pairs of plates. Every plate in the model moves relative to some other plate or is the “fixed reference frame” relative to which some other plates move. In general, large plates are described relative to other large plates, and smaller plates are described relative to their nearest large plates. This tree-like structure is called the rotation hierarchy. Even if there is actually no relative motion between some pair of plates for some period of time, the plates may nevertheless be related by the identity rotation (a rotation of zero angle, which effects no change of position or orientation).

The finite rotation that displaces a plate from its present-day position to its reconstructed position at some instant in the past is called a __total reconstruction pole__ (Cox and Hart, 1986) (henceforth TRP). A TRP is associated with a fixed plate, a moving plate, and a particular instant in geological time:


An __equivalent rotation__ is the rotation of a plate relative to the anchored plate.

A __relative rotation__ is the rotation of one plate relative to another plate (as opposed to the anchored plate).

A __total rotation__ is a rotation at a time in the past relative to present day (0Ma). In other words from present day to a past time.

A __stage rotation__ is a rotation at a time in the past relative to another time in the past.

A __finite rotation__ represents the motion of a plate (relative to another plate) on the surface of the globe over a period of geological time.

See:

http://www.gplates.org/docs/pygplates/pygplates_foundations.html#pygplates-foundations-plate-reconstruction-hierarchy

# Rotations

```
input_rotation_filename = 'Data/Seton_etal_ESR2012_2012.1.rot'
rotation_model = pygplates.RotationModel(input_rotation_filename)
finite_rotation = rotation_model.get_rotation(40.1,801,0,802)
pole_lat,pole_lon,pole_angle = finite_rotation.get_lat_lon_euler_pole_and_angle_degrees()
```

 Note that in the call to the function 'rotation_model.get_rotation', there are four input parameters

1. the time to reconstruct to (40.1 Ma)
2. the moving plate id (801 for Australia)
3. the time to reconstruct from (0 for present day)
4. the fixed plate id (802 for Antarctica)

To get successive stage rotations (a stage rotation sequence) just loop through a series of times, and generate `rotation_model.get_rotation()`

__operator overloading__

Note there seems to be some operator overloading for rotations :

```
for feature in features:
    if feature.get_reconstruction_plate_id() == 701:
        polygon = feature.get_geometry()
        reconstructed_point = rotation * polygon.get_boundary_centroid()
```

`rotation` is a pygplates.FiniteRotation 

`polygon.get_boundary_centroid()` is a pygplates.PointOnSphere 

Does this work for Polygons? It seems to work fine: 

```
rotation * polygon
<pygplates.PolygonOnSphere at 0x11558fe50>
```

Multiple rotations can be composed this way

`geometry_final = R2 * R1 * geometry_initial`

which is the same as:

`geometry_final = pygplates.FiniteRotation.compose(R2, R1) * geometry_initial`

​	

## Velocities

The following example is worth taking some time to unpack:

http://www.gplates.org/docs/pygplates/sample-code/pygplates_calculate_velocities_by_plate_id.html

To get the velocity on an object at a time past, we have to reconstruct it to it's past position, for which we need a __total reconstruction__:

` # Get the rotation of plate 'domain_plate_id' from present day (0Ma) to 'reconstruction_time'.    equivalent_total_rotation = rotation_model.get_rotation(reconstruction_time, domain_plate_id)`

Then we can put the feature in its correct place:

```
# Reconstruct the geometry to 'reconstruction_time'.
reconstructed_geometry = equivalent_total_rotation * geometry
reconstructed_points = reconstructed_geometry.get_points()
```

To get the velocity at the time in the past, we want to specify an equivalent rotation (i.e. a rotation with a dt that it as small as it can be...)

```
# This is from 'reconstruction_time + delta_time' to 'reconstruction_time' on plate 'domain_plate_id'.
velocity_vectors = pygplates.calculate_velocities(reconstructed_points, equivalent_stage_rotation, delta_time)
```

In Simon Williams pygplates recipes, he's packaged some of this stuff up into a function:

` GetVelocityForPoint(velocity_point,moving_plate,fixed_plate,velocity_type='MagAzim'):`

# topologies

Within GPlates, the plate polygons are 'resolved' by piecing together the various boundary segments from individual geometries and removing any excess line sections. 

```
rotation_filename = 'Data/Seton_etal_ESR2012_2012.1.rot'
input_topology_filename = 'Data/Seton_etal_ESR2012_PP_2012.1.gpmlz'
topology_features = pygplates.FeatureCollection(input_topology_filename)
rotation_model = pygplates.RotationModel(rotation_filename)
# Specify time at which to create resolved topological plate polygons
time=100.
resolved_topologies = []
pygplates.resolve_topologies(topology_features, rotation_model, resolved_topologies, time)
```

__the number of plates is the length of the resolved topologies object__

```
# the number of plates is the length of the resolved topologies object 
num_plates = len(resolved_topologies)
print 'Number of Plates is ',num_plates

```

for topology in resolved_topologies:

    for topology in resolved_topologies:
        # Get the plate area - note we use the built in pygplates Earth radius to get 
        plate_geometry = topology.get_resolved_geometry()
For some cases, there is an extra level between the resolved topologies object and the plate:

`plate_feature = topology.get_resolved_feature()`

As well as looking at closed plate polygons, the resolved topologies can also be analysed to extract line segments that are attributed with the type of plate boundary - typically these are either subduction zone segments or ridge-transform boundary segments, with some other types also appearing (e.g. 'inferredPaleoBoundary').



There seems to be three types of object that result from Resoving::

Classes resulting from resolving topological features at a particular reconstruction time.
`pygplates.ResolvedTopologicalLine` 	The geometry of a topological line feature resolved to a geological time.
`pygplates.ResolvedTopologicalBoundary` 	The geometry of a topological boundary feature resolved to a geological time.
`pygplates.ResolvedTopologicalNetwork` 	The geometry of a topological network feature resolved to a geological time.









## Reconstructed vs resolved

```
 pygplates.reconstruct(reconstructable_features, rotation_model, reconstructed_geometries, reconstruction_time[, anchor_plate_id=0][, **output_parameters])

    Reconstruct regular geological features, motion paths or flowlines to a specific geological time.
```

Does reconstruction only apply to features in their present-day geometry? Their is no "from time" in the arg list, which makes me think this is the case .

the basic 'pygplates.reconstruct' function, which is suitable for  reconstructing all the features in a collection for a single 
reconstruction time.

# Features and geoms

Features generally combine geometries and a number of other attributes, such as plate Id, valid times etc.  There 

Feature collections

`MultiPointOnSphere`
`pygplates.PolygonOnSphere`



```
print(typrint(type(ridge_feat.get_geometry()))
print(type(ridge_feat.get_geometry().get_segments()))
***
<class 'pygplates.PolylineOnSphere'>
<class 'pygplates.PolylineOnSphereArcsView'>
```

What is a Segment? 

* a segment is a pygplates.GreatCircleArc, That is - an implied GCA between two points. 

It seems like lots of features, like MORS are naturally segmented - into, i.e. ridges and transforms (although they are not tagged as such, hence the functionality to identiify these in PTT)



## functions

`create_motion_path`

```stageData =pygplates.get_equivalent_stage_rotation(fromTimeRotation,toTimeRotation,moving_plate)```

`pole,angle = stageData.get_euler_pole_and_angle()`

`pole = pygplates.convert_point_on_sphere_to_lat_lon_point(pole)`

`pygplates.reconstruct`:
  Reconstruct regular geological features, motion paths or flowlines to a specific geological time.


## Reference frames

the absolute reference frames have three-digit plate IDs which begin with a
“0.”1

## Questions

