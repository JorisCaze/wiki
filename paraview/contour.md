# Plot contour of an interface 

Let say we want to track the Rayleigh--Taylor instability by volume fraction, we have to look at the 2D colored plot.

Since the interface is diffused it can be hard to explicitly define it.

`ParaView` can draw for you the contour at a given value. 

To do this we need to apply the following filters:

- clean to grid
- merge blocks
- cell data to point data
- contour applied on volume fraction 

Note that this process works on `ParaView 5.6.0`, `5.7` but not `5.8`.