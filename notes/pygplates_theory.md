# PyGplates

## Absolute plate motions

A good paper on reference frames is Williams et al. __Absolute plate motions since 130 Ma constrained by subduction zone kinematics. _They make the distinction between relative plate motion models RPM. which describes the relative motions of the plates (like most EarthByte stuff, they use Seton 2012), and the Absolute plate model (APM), which is basically just the motion of Africa in the Seton hierarchy. 

For times older than 83 Ma, the Pacific plate is surrounded by subduction zones and cannot be linked to Africa
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

## How rotations look in pygplates

```python
input_rotation_filename = 'Data/Seton_etal_ESR2012_2012.1.rot'
rotation_model = pygplates.RotationModel(input_rotation_filename)
finite_rotation = rotation_model.get_rotation(40.1,801,0,802)
pole_lat,pole_lon,pole_angle = finite_rotation.get_lat_lon_euler_pole_and_angle_degrees()
```

In the call to the function 'rotation_model.get_rotation', there are four input parameters

1. the time to reconstruct to (40.1 Ma)
2. the moving plate id (801 for Australia)
3. the time to reconstruct from (0 for present day)
4. the fixed plate id (802 for Antarctica)

So this meets the criteria of a Finite Rotation, but is it also a Relative Rotation. 	


## Reference frames

the absolute reference frames have three-digit plate IDs which begin with a “0.”



