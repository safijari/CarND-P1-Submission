## Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflection on the work done in this written report

[//]: # (Image References)
[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

#### Pipeline steps:

1. I resize the incoming image to hard coded values of 960 x 540 (the size of the initial test images and 
videos). This was in response to the challenge video which would have worked fine using my initially tuned 
parameters had it been this size.

2. I gaussian blur the resized image and then run it through Canny (I don't greyscale it because I think 
opencv's canny does that internally) and then mask the resulting canny image to a triangular-ish polygon. 
The parameters for these were tuned by using `ipywidgets.interactive`.

3. I apply `hough_lines` to the resulting edge image with yet again hand tuned parameters.

#### How I modified `draw_lines()`:
This gets a bit complicated. I wrapped most of the functions I use in a class so I could keep state. I wanted
to save past lines so I could use them as a temporal filtering step and make the lines on the video look 
smooth. Then inside `draw_lines`, I compute slope for each line and discard any that have an absolution slope
smaller than a hand selected threshold. I then take a median of the slopes of the positive sloped lines and
the negative sloped lines (the intuition being that for the scenario we're dealing with the car must always)
be in the middle of the road. I then further filter lines by discarding any that don't have a slope within
0.05 of the medians. I then take the resulting lines and take another median for each set of lines to give me
a slope and an intercept. I then update the `line_params` member of the wrapping class and draw the full line
by setting the `y` values as free variables (at the apex of my mask polygon) and solving for `x`.

I also modified the coloring to reflect the slope somewhat, adding or removing a green component based on the 
sign and value of the slope.


### 2. Identify potential shortcomings with your current pipeline

1. If the car is not roughly in the middle of the road, this pipeline won't work.
2. If the lane lines are not very visible, this pipeline won't work.
3. If you changed the camera location/fov significantly, this would break.
4. It may behave weirdly with quick steering movements since there is a moving average step.
5. I would not sit in or put a loved one in a car using this algorithm!

### 3. Suggest possible improvements to your pipeline

1. Need a better way to reject outliers, median works well enough but it will likely fail in noisier
environments.
2. Some form of tracking of the horizon/vanishing point as suggested by a cohort could help.
3. Perhaps something like a Kalman filter for filtering the lines termporally would be better to have, 
with a reasonable model of the lane and the car and inputs from other sensors/feature tracking in the video
would be a great help.
