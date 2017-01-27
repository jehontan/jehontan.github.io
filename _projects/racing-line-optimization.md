---
layout: post
title: Racing Line Optimization
date: 2016-12-22 04:07:00 -0500
categories:
short_description: Vehicle trajectory optimization using genetic algorithm.
image_preview:
---

In motorsports one often hears about the racing line. This is a hypothetical
optimal trajectory that the vehicle should follow in order to maximize speed.
Since I watch a lot of Top Gear, I thought I might try my hand at finding the
optimal racing line for a given racetrack.

## Problem Definition and Assumptions

Given a racetrack, the aim is to find the trajectory that minimizes the time
taken to travel around the track once. The track is assumed to be in the plane,
i.e. 2D.

To simplify the problem, the vehicle is modelled as a particle, hence all
vehicle dynamics are ignored (roll, complex tire side-slip conditions,
aerodynamics, etc.). The vehicle is simply assumed to have a fixed top speed
\\(v_{max}\\), maximum acceleration \\(a_{t,max}\\) and maximum braking acceleration
\\(a_{b,max}\\).

The side-slip condition used was a maximum centripetal acceleration threshold.
The maximum velocity possible for any given point along a trajectory was then
given by the relation

$$a_c = \frac{v^2}{\rho} $$

where \\(a_c\\) is the centripetal acceleration, \\(v\\) is the velocity and \\(\rho\\)
is the radius of curvature of the trajectory at the point.

## Track Information

Track information was extracted from a picture of the track by applying MATLAB's
[bwboundaries()](https://www.mathworks.com/help/images/ref/bwboundaries.html) function
to obtain the inner and outer boundaries of the track. The figure below shows the
result of this process, with the inner and outer boundaries of the track highlighted
in red and blue respectively.

<img src="{{ "images/track_boundaries.png" | relative_url }}" width="400">

## Trajectory Model

The trajectory of the vehicle was represented using a number of control points around
the track that were interpolated to form the trajectory. This was done in two ways.

The initial idea was to use a 4th-order B-spline, in order to be able to extract
smooth curvature information at any point along the track. However it was later
found that computing the spline and extracting curvature information was fairly
slow. For that reason a second method using linear interpolation with a higher
density of control points was used instead.

### Control point modification

To vary the trajectory, the control points needed to be modified. Using a B-spline,
one can modify the position of the control points or knots, and even repeat control
points or knots to increase their "weight" in order to modify the resultant trajectory.
Using a linear interpolation, one can only modify the position of the control points,
significantly reducing the complexity of the task.

In order for the trajectory to be useful, the positions of the control points need
to be constrained, otherwise a naïve optimization algorithm might converge on a result
where all the control points are collocated. Two methods of constraining the positions
of the control points were explored.

The first method placed control points evenly along the track and confined their
positions to lie along evenly-spaced line segments perpendicular to the inner and
outer boundaries. The position of each control point on its corresponding line segment
can then be done using a parameter \\(\delta\\) that varies from 0 to 1, with 0
corresponding to the inner boundary and 1 to the outer boundary. A vector of these
\\(\delta\\) values then represent a trajectory. The below figure illustrates this
method.

<img src="{{ "images/delta_method.png" | relative_url }}" width="400">

While this representation is easy, it has the disadvantage of not being able to handle
tight corners very well. This is due to the fixed spacing of the control points
potentially causing no control points to fall near the apex of the corner.

To overcome this, the second method constrains the positions of the control points
within boxes rather than on lines. However an additional check is required to ensure
that the control points do not lie outside the track. The below figure illustrates
this method.

<img src="{{ "images/free_method.png" | relative_url }}" width="400">

## Minimum Curvature

Intuitively then, for a fast lap time around the track, one would seek to maximize
velocity, and hence minimize the curvature \\(\kappa = 1/\rho\\).

As mentioned above, the initial idea was to use a B-spline to represent the trajectory
which would allow one to obtain the an exact curvature, however that proved too
computationally expensive. Instead a linear interpolation between control points was
used and the degree of curvature estimated using the angle formed by 3 consecutive
control points, with a smaller angle indicating higher curvature. One can then minimize
the sum of the inverse of the angles, \\(1/\theta\\) to minimize the curvature.

### Non-causality

Since it takes a point before and after in order to compute the curvature at a given
point, it is not possible to modify the positions of each point individually to
obtain the lowest curvature. Furthermore because the next position of the next control
point affects the curvature at the current point, the problem is non-causal, i.e.
one cannot determine the best position based on the positions of past points alone.

Hence one cannot speak of the fitness of the position of a particular control point
without being given the whole trajectory. For this reason we decided to use a
stochastic optimization method, specifically a genetic algorithm, for the problem.

## Minimum Time

A minimum curvature solution, while a step in the right direction, is not guaranteed
to give the fastest lap time. This is because there is a trade-off between lateral
acceleration (which must be below the side-slip condition) and trajectory length.
Using a convex track as an example, the shortest possible trajectory is the inner
boundary, but the minimum curvature solution would favour the outer boundary.

Hence depending on the performance of the vehicle, using aggressive acceleration
and braking it may be possible to make tighter turns and still obtain a faster lap
time. Directly optimizing for lap time is thus the more correct approach.

### Velocity Profile Generation

In order to obtain the lap-time for a given trajectory it is necessary to generate
an optimum velocity profile for the trajectory.

For every point on the trajectory one can compute the fastest possible velocity attainable
while still fulfilling the side-slip condition. This is simply obtained from the
relation

$$v = \sqrt{\frac{a_{c,max}}{\kappa}}$$

where \\(a_{c,max}\\) is the lateral acceleration at which side-slip occurs.

<img src="{{ "images/v_curvature.png" | relative_url }}" width="600">

Next the maximum braking condition is applied, starting from the end of the trajectory
to the start, resulting in the following velocity profile:

<img src="{{ "images/v_braking.png" | relative_url }}" width="600">

The maximum acceleration condition is then applied, from the start of the trajectory
to the end:

<img src="{{ "images/v_acceleration.png" | relative_url }}" width="600">

And lastly the maximum velocity condition is applied which saturates the velocity
at the maximum velocity.

### Non-causality

As with the case of the minimum curvature problem the minimum time problem is
non-causal due to its reliance on curvature information. Furthermore the velocity
profile generation requires future maximum velocities to be known in order to obtain
apply the maximum braking condition and obtain the velocity at the current point.

The genetic algorithm approach was thus also applied to the minimum time problem.

## Results and Discussion

I won't go into the details of the implementation, but in brief the optimization
was performed using MATLAB's genetic algorithm function.

The below figure shows the result obtained for the minimum curvature problem. As
you can see it's exactly the result one would expect for a minimum curvature trajectory.

<img src="{{ "images/results_curvature.png" | relative_url }}" width="400">

This next figure shows the result for the minimum time problem, with the trajectory
coloured red for low velocity and green for high velocity. The vehicle starts in the top
left and travels counter-clockwise. Some cool behaviour can be seen, such as a
double-apex behaviour in the top right corner (compare to [this](https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Double_Apex.jpg/1024px-Double_Apex.jpg)),
which is not seen in the naïve minimum curvature solution.

<img src="{{ "images/results_time.png" | relative_url }}" width="400">

## Conclusion

The optimum racing line trajectory around a racetrack was found using two approaches,
one using minimum curvature and one using minimum lap-time. Both approaches yielded
optimization problems that were non-causal and demanded the use of a stochastic
optimization technique. The genetic algorithm was used on both problems. The minimum
curvature solution is simplistic, but provides a good "first-guess". The minimum time
approach yielded results consistent with expectations and real-life racing behaviour.

## Acknowledgements

This project was done in collaboration with John Papayanopoulos. Our full paper is
available [here]({{ "racing-line-optimization.pdf" | relative_url }})
