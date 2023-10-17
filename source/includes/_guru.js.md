# Guru.js

Guru.js is the Javascript framework that allows you to tell
the Guru AI platform how to process your video. The same code
runs on both the server-side API and on-device, allowing you to
write once and run anywhere.

Before you can begin writing Guru.js, you will need to create a Schema.
A `Schema` is a configuration that holds all the information Guru
needs to process a video. You can create a new Schema from
the [Guru Console](https://console.getguru.fitness/schemas/new).
We recommend starting from a template to get up and running faster.

A Schema holds three separate Guru.js functions:

1. **Process** - This function defines what AI operations should be carried out on each frame.
1. **Render** - This function defines what visual rendering should be done on each frame of the video. It can be used to visualize the output of Guru's AI platform. It is optional.
1. **Analyze** - This function runs once on the output of all the frames. It is used to perform final analysis on the video, such as counting the number of repetitions of a movement.

Guru will call your `Process` function for individual frames within the video. It will then
pass the result of the AI operations to your `Render` function, so that you can modify
the display of the output video. Finally, it will pass the collection of `Process` results
to your `Analyze` function, so that you can perform aggregate analysis of the video. 

We will now go over each function in more detail.

## Process

> The Process function allows you to perform AI operations on a frame within the video.

```javascript
/**
 * @param {Frame} frame - The frame to process
 * @return {*} - The output of processing for this frame.
 */
async function processFrame(frame) {
  return {};
}

class Frame {

  /**
   * Find objects of a specific type within the video.
   *
   * @param {(string|Array.<string>)} objectTypes - The type of the object to find.
   *    Can either be a string, in which case objects of a single type will be found, or an array of strings, in which case multiple object types will be found.
   * @param {boolean} keypoints - Flag indicating whether to include keypoints in the results. Defaults to true.
   * @return {Array.<FrameObject>} A list of FrameObject instances matching the given criteria.
   */
  async findObjects(objectTypes, {keypoints = true} = {});
}
```

The Process function is responsible for performing AI operations on the frame,
such as object detection and pose estimation, and returning the results
of those operations. The function accepts a single argument of type `Frame`. 
This object provides an easy-to-use interface to the Guru AI Platform.

Refer to [Appendix A](#appendix-a) for a definition of the common types used by `Frame`.

The output of the `Process` function will then be provided as input to the
`Render` and `Analyze` functions documented below.

> Example implementation that finds all of the people in the frame and outputs their location information

```javascript
async function processFrame(frame) {
	const objects = await frame.findObjects("person");

	return {people: objects};
}
```

## Render

> The Render function allows you to modify the FrameCanvas with the output of the Process function.

```javascript
/**
 * @param {FrameCanvas} frameCanvas - A canvas for the current frame, on which draw operations can be made.
 * @param {*} processResult - The output of the Process function for this frame.
 */
function renderFrame(frameCanvas, processResult) {
}

/**
 * The canvas for a frame, onto which draw operations can be made. This is passed as input to renderFrame().
 */
class FrameCanvas {

  /**
   * Draw a box with the given color around this object's location onto the frame.
   *
   * @param {FrameObject} object - The object around which the box will be drawn.
   * @param {Color} color - The color of the box.
   * @param {number} width - The width of the box's border, in pixels. Defaults to 5.
   * @return {FrameCanvas} This FrameCanvas, that can be used to chain calls.
   */
  drawBoundingBox(object, color, width = 2);

  /**
   * Draws a circle on the canvas.
   *
   * @param {Position} position - The position of the center of the circle.
   * @param {number} radius - The radius of the circle, in pixels.
   * @param {Color} color - The color of the circle.
   * @param {boolean} filled - True if the circle should be filled in. Default true.
   * @param {number} width - If not filled, then this is the width of the circle boundary in pixels. Default 2.
   * @param {number} alpha - Optional, how transparent the circle should be. 0 is invisible, 1 is fully visible. Default is 1.
   */
  drawCircle(position, radius, color, {
    filled = true,
    width = 2,
    alpha = 1.0,
  } = {});

  /**
   * Draws a line between two points on the canvas.
   *
   * @param {Position} from - The position to draw from.
   * @param {Position} to - The position to draw to.
   * @param {Color} color - The color of the line.
   * @param {number} width - Optional, the width of the line in pixels. Default 2.
   * @param {number} alpha - Optional, how transparent the line should be. 0 is invisible, 1 is fully visible. Default is 1.
   */
  drawLine(from, to, color, {
    width = 2,
    alpha = 1.0,
  } = {});

  /**
   * Draws a rectangle on the canvas. The rectangle may have a background color, or be transparent.
   *
   * @param {Position} topLeft - The position of the top-left corner of the rectangle.
   * @param {Position} bottomRight - The position of the bottom-right corner of the rectangle.
   * @param {Color} borderColor - Optional, the color of border of the rectangle. Either this or backgroundColor must be present.
   * @param {Color} backgroundColor - Optional, the color of background of the rectangle. If omitted then the background will be transparent. Either this or backgroundColor must be present.
   * @param {number} width - Optional, the width of the border in pixels. Default 2.
   * @param {number} alpha - Optional, how transparent the rectangle should be. 0 is invisible, 1 is fully visible. Default is 1.
   */
  drawRect(topLeft, bottomRight, {
    borderColor = undefined,
    backgroundColor = undefined,
    width = 2,
    alpha = 1.0,
  } = {});

  /**
   * Draw the skeleton for the given object onto the frame. Note that the object must have its keypoints
   * inferred in order to draw the skeleton.
   *
   * @param {FrameObject} object - The object whose skeleton will be drawn onto the canvas.
   * @param {Color} lineColor - The color of the lines connecting the keypoints in the skeleton.
   * @param {Color} keypointColor - The color of the circles representing the joint keypoints in the skeleton.
   * @param {number} lineWidth - The width, in pixels, of the lines connecting the keypoints. Defaults to 5.
   * @param {number} keypointRadius - The radius, in pixels, of the circles representing the keypoints. Defaults to 5.
   */
  drawSkeleton(object, lineColor, keypointColor, lineWidth = 2, keypointRadius = 5);

  /**
   * Draws text at a specific location on the canvas.
   *
   * @param {string} text - The text to draw.
   * @param {Position} position - The location to draw at. 0,0 is the top-left corner.
   * @param {Color} color - The color of the text.
   * @param {number} maxWidth - Optional, the maximum width of the text in pixels, after which it will wrap. Default 1000.
   * @param {number} fontSize - Optional, the size of the font. Default 24.
   * @param {number} padding - Optional, the amount of padding to apply to the location of the text from its location. Default 0.
   * @param {number} alpha - Optional, how transparent the font should be. 0 is invisible, 1 is fully visible. Default is 1.
   */
  drawText(text, position, color, {
    maxWidth = 1000,
    fontSize = 24,
    padding = 0,
    alpha = 1.0,
  } = {});
```

The Render function receives two arguments: one of type `FrameCanvas`, defined below, 
and a second which is the output of the `Process` function for that frame. 
Your implementation can use the `FrameCanvas` object to modify the appearance of the 
output video, using information from the output of `Process`.

Refer to [Appendix A](#appendix-a) for a definition of the common types used by `FrameCanvas`.

> Example implementation that draws a red box around each object found in the frame

```javascript
function renderFrame(frameCanvas, processResult) {
	const objects = processResult.objects;
	objects.forEach(object => {
		frameCanvas.drawBoundingBox(object, new Color(255, 0, 0));
	});
}
```

## Analyze

> The Analyze function allows you to operate on the output of multiple Process functions and return the analysis for the video.

```javascript
/**
 * @param {Array.<FrameResult>} frameResults - An array of the outputs of calls to the Process function.
 * @return {*} - The analysis result for this video
 */
async function analyzeVideo(frameResults) {
   return {};
}

/**
 * @typedef {Object} FrameResult
 * @property {number} frameIndex - The index of the frame within the video.
 * @property {number} timestampMs - The timestamp of the frame in milliseconds.
 * @property {*} returnValue - The value that was returned from the Process function for this frame.
 */
```

> Example implementation that reduces the FrameResults to an array of unique object types found across the video.

```javascript
async function analyzeVideo(frameResults) {
	const frameObjectTypes = frameResults.map((frameResult) => {
		return frameResult.returnValue.objects.map((object) => {
			return object.objectType;
		});
	}).flat();

	return Array.from(new Set(frameObjectTypes));
}
```

> Example implementation that counts reps.

```javascript
async function analyzeVideo(frameResults) {
	const personId = frameResults.objectIds("person")[0];
	const personFrames = frameResults.objectFrames(personId);
	const reps = MovementAnalyzer.repsByKeypointDistance(personFrames, Keypoint.rightHip, Keypoint.rightAnkle);
	return {
		"reps": reps
	};
}
```

The Analyze function receives one argument, an Array of `FrameResult` objects. 
This function can perform any kind of analysis required on the results of each Frame
to perform complex reasoning. It could, for example, count the number of repetitions
of a particular movement, or de-duplicate the names of unique objects found across the
entire video.

The output of this function will be the analysis result of the video. It will be accessible
from the [Get Analysis](https://docs.getguru.fitness/#get-analysis) endpoint when using
server-side processing, or as the output of the SDK when performing on-device processing.

The Array of `FrameResult`s is augmented by a number of useful methods for performing analysis:

Method | Description
------ | ------- |
`frameResults.objectFrames(objectId)` | Given the ID of an object, fetch the `FrameObject`s that describe its movement throughout the video.
`frameResults.objectIds(objectType)` | Given a type of object (e.g. `person`), fetch the IDs of each instance found of that type in the video.
`frameResults.resultArray()` | Return an array of the raw results from each call to your Process code.
 
Guru.js also provides a library of domain-specific analysis functions to help perform common operations.

### MovementAnalyzer

This Analyzer provides methods related to human movement. It is useful for building fitness or sport-focused apps.

Method | Parameters | Description
------ | ------- | ------- |
`repsByKeypointDistance` | <ol><li>personFrames</li><li>keypoint1</li><li>keypoint2</li><li>{keypointsContract: true, threshold: 0.2, smoothing: 2.0, ignoreStartMs: null, ignoreEndMs: null}</li></ol> | <p>Given the frames of a person, find repetitions of a movement defined by the movement between two keypoints. For example, you could use `Keypoint.rightHip` and `Keypoint.rightAnkle` to count the number of squat reps in a video.</p><p>`keypointsContract` is an optional parameter that indicates whether the distance between the keypoints will contract during a rep. Set to false if the keypoint distance will instead expand. Defaults true.</p><p>`threshold` and `smoothing` control how reps are identified, tuning them may give better results for your movement.</p><p>`ignoreStartMs` and `ignoreEndMs` allow you to exclude time at the start or end of the video for rep counting. If null, then Guru will attempt to guess the times.</p>
`personMostlyFacing` | <ol><li>personFrames</li></ol> | Given the frames of a person, determine which direction they were mostly facing during the course of the video. Returns a value from the `ObjectFacing` enum.
`personMostlyStanding` | <ol><li>personFrames</li></ol> | Given the frames of a person, determine whether they were standing for the majority of the video. Returns a boolean.

`ObjectFacing` Enum | Description
------ | ------- |
Away | Object facing away from the camera.
Down | Object facing towards the bottom of the frame.
Left | Object facing the left-side of the frame.
Right | Object facing the right-side of the frame.
Toward | Object facing towards the camera.
Unknown | Object direction unknown.
Up | Object facing towards the top of the frame.

## Appendix A
```javascript
/**
 * A two-dimensional box, indicating the bounds of something.
 *
 * @typedef {Object} Box
 * @property {Position} top_left - The top-left corner of the box.
 * @property {Position} bottom_right - The bottom-right corner of the box.
 */

/**
 * A colour, represented as an RGB value. Valid values for each are >= 0 and <= 255.
 *
 * @typedef {Object} Color
 * @property {number} r - The amount of red in the color.
 * @property {number} g - The amount of green in the color.
 * @property {number} b - The amount of blue in the color.
 */

/**
 * A single object present within a particular frame or image.
 *
 * @typedef {Object} FrameObject
 * @property {string} objectType - The type of the object.
 * @property {Box} boundary - The bounding box of the object, defining its location within the frame.
 * @property {Object.<string, Position>} keypoints - A map of the name of a keypoint, to its location within the frame.
 */

/**
 * The two-dimensional coordinates indicating the location of something.
 *
 * @typedef {Object} Position
 * @property {number} x - The x coordinate of the position.
 * @property {number} y - The y coordinate of the position.
 * @property {number} confidence - The confidence Guru has of the accuracy of this position. 0.0 implies no confidence, 1.0 implies complete confidence.
 */
```

Documented here are common types that are used across each of the
`Process`, `Render`, and `Analyze` functions.
