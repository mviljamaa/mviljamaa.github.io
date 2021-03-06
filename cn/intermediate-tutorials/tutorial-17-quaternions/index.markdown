---
layout: page
status: publish
published: true
title: 'Tutorial 17 : Rotations'
date: '2012-08-22 14:12:14 +0200'
date_gmt: '2012-08-22 14:12:14 +0200'
categories: [tuto]
order: 90
tags: []
language: cn
---
<p>This tutorial goes a bit outside the scope of OpenGL, but nevertheless tackles a very common problem: how to represent rotations ?</p>
<p>In Tutorial 3 - Matrices, we learnt that matrices are able to rotate a point around a specific axis. While matrices are a neat way to transform vertices, handling matrices is difficult: for instance, getting the rotation axis from the final matrix is quite tricky.</p>
<p>We will present the two most common ways to represent rotation: Euler angles and Quaternions. Most importantly, we will explain why you should probably use Quaternions.</p>
<p><a href="http://www.opengl-tutorial.org/wp-content/uploads/2012/08/tuto17.png"><img class="alignnone size-large wp-image-786" title="tuto17" src="http://www.opengl-tutorial.org/wp-content/uploads/2012/08/tuto17-1024x793.png" alt="" width="640" height="495" /></a></p>
<h1>Foreword: rotation VS orientation</h1><br />
While reading articles on rotations, you might get confused because of the vocabulary. In this tutorial:</p>
<ul>
<li>An orientation is a state: "the object's orientation is..."</li>
<li>A rotation is an operation: "Apply this rotation to the object"</li><br />
</ul><br />
That is, when you&nbsp;<em>apply a rotation</em>, you <em>change the orientation</em>. Both can be represented with the same tools, which leads to the confusion. Now, let's get started...</p>
<h1>Euler Angles</h1><br />
Euler angles are the easiest way to think of an orientation. You basically store three rotations around the X, Y and Z axes. It's a very simple concept to grasp. You can use a vec3 to store it:</p>
<pre class="brush:cpp">vec3 EulerAngles( RotationAroundXInRadians, RotationAroundYInRadians, RotationAroundZInRadians);</pre><br />
These 3 rotations are then applied successively, usually in this order: first Y, then Z, then X (but not necessarily). Using a different order yields different results.</p>
<p>One simple use of Euler angles is setting a character's orientation. Usually game characters do not rotate on X and Z, only on the vertical axis. Therefore, it's easier to write, understand and maintain "float direction;" than 3 different orientations.</p>
<p>Another good use of Euler angles is an FPS camera: you have one angle for the heading (Y), and one for up/down (X). See common/controls.cpp for an example.</p>
<p>However, when things get more complex, Euler angle will be hard to work with. For instance :</p>
<ul>
<li>Interpolating smoothly between 2 orientations is hard. Naively interpolating the X,Y and Z angles will be ugly.</li>
<li>Applying several rotations is complicated and unprecise: you have to compute the final rotation matrix, and guess the Euler angles from this matrix</li>
<li>A well-known problem, the "Gimbal Lock", will sometimes block your rotations, and other singularities which will flip your model upside-down.</li>
<li>Different angles make the same rotation ( -180&deg; and 180&deg;, for instance )</li>
<li>It's a mess - as said above, usually the right order is YZX, but if you also use a library with a different order, you'll be in trouble.</li>
<li>Some operations are complicated: for instance, rotation of N degrees around a specific axis.</li><br />
</ul><br />
Quaternions are a tool to represent rotations, which solves these problems.</p>
<h1>Quaternions</h1><br />
A quaternion is a set of 4 numbers, [x y z w], which represents rotations the following way:</p>
<pre class="brush:cpp">// RotationAngle is in radians<br />
x = RotationAxis.x * sin(RotationAngle / 2)<br />
y = RotationAxis.y * sin(RotationAngle / 2)<br />
z = RotationAxis.z * sin(RotationAngle / 2)<br />
w = cos(RotationAngle / 2)</pre><br />
RotationAxis is, as its name implies, the axis around which you want to make your rotation.</p>
<p>RotationAngle is the angle of rotation around this axis.</p>
<p><a href="http://www.opengl-tutorial.org/wp-content/uploads/2012/08/quaternion.png"><img class="alignnone  wp-image-762 whiteborder" title="quaternion" src="http://www.opengl-tutorial.org/wp-content/uploads/2012/08/quaternion.png" alt="" width="305" height="361" /></a></p>
<p>So essentially quaternions store a <em>rotation axis</em> and a <em>rotation angle</em>, in a way that makes combining rotations easy.</p>
<h2>Reading quaternions</h2><br />
This format is definitely less intuitive than Euler angles, but it's still readable: the xyz components match roughly the rotation axis, and w is the acos of the rotation angle (divided by 2). For instance, imagine that you see the following values in the debugger: [ 0.7 0 0 0.7 ]. x=0.7, it's bigger than y and z, so you know it's mostly a rotation around the X axis; and 2*acos(0.7) = 1.59 radians, so it's a rotation of 90&deg;.</p>
<p>Similarly, [0 0 0 1] (w=1) means that angle = 2*acos(1) = 0, so this is a <em>unit quaternion</em>, which makes no rotation at all.</p>
<h2>Basic operations</h2><br />
Knowing the math behind the quaternions is rarely useful: the representation is so unintuitive that you usually only rely on utility functions which do the math for you. If you're interested, see the math books in the <a title="Useful Tools &amp; Links" href="http://www.opengl-tutorial.org/miscellaneous/useful-tools-links/">Useful Tools &amp; Links</a> page.</p>
<h3>How do I create a quaternion in C++ ?</h3></p>
<pre class="brush:cpp">// Don't forget to #include <glm/gtc/quaternion.hpp> and <glm/gtx/quaternion.hpp></p>
<p>// Creates an identity quaternion (no rotation)<br />
quat MyQuaternion;</p>
<p>// Direct specification of the 4 components<br />
// You almost never use this directly<br />
MyQuaternion = quat(w,x,y,z); </p>
<p>// Conversion from Euler angles (in radians) to Quaternion<br />
vec3 EulerAngles(90, 45, 0);<br />
MyQuaternion = quat(EulerAngles);</p>
<p>// Conversion from axis-angle<br />
// In GLM the angle must be in degrees here, so convert it.<br />
MyQuaternion = gtx::quaternion::angleAxis(degrees(RotationAngle), RotationAxis);</pre></p>
<h3>How do I create a quaternion in GLSL ?</h3><br />
You don't. Convert your quaternion to a rotation matrix, and use it in the Model Matrix. Your vertices will be rotated as usual, with the MVP matrix.</p>
<p>In some cases, you might actually&nbsp;want to use quaternions in GLSL, for instance if you do skeletal animation on the GPU. There is no quaternion type in GLSL, but you can pack one in a vec4, and do the math yourself in the shader.</p>
<h3>How do I convert a quaternion to a matrix ?</h3></p>
<pre class="brush:cpp">mat4 RotationMatrix = quaternion::toMat4(quaternion);</pre><br />
You can now use it to build your Model matrix as usual:</p>
<pre class="brush:cpp">mat4 RotationMatrix = quaternion::toMat4(quaternion);<br />
...<br />
mat4 ModelMatrix = TranslationMatrix * RotationMatrix * ScaleMatrix;<br />
// You can now use ModelMatrix to build the MVP matrix</pre></p>
<h3></h3></p>
<h1>So, which one should I choose ?</h1><br />
Choosing between Euler angles and quaternions is tricky. Euler angles are intuitive for artists, so if you write some 3D editor, use them. But quaternions are handy for programmers, and faster too, so you should use them in a 3D engine core.</p>
<p>The general consensus is exactly that: use quaternions internally, and expose Euler angles whenever you have some kind of user interface.</p>
<p>You will be able to handle all you will need (or at least, it will be easier), and you can still use Euler angles for entities that require it ( as said above: the camera, humanoids, and that's pretty much it) with a simple conversion.</p>
<h1>Other resources</h1></p>
<ol>
<li>The books on&nbsp;<a title="Useful Tools &amp; Links" href="http://www.opengl-tutorial.org/miscellaneous/useful-tools-links/">Useful Tools &amp; Links</a>&nbsp;!</li>
<li>As old as it can be, Game Programming Gems 1 has several awesome articles on quaternions. You can probably find them online too.</li>
<li>A <a href="http://www.essentialmath.com/GDC2012/GDC2012_JMV_Rotations.pdf">GDC presentation</a> on rotations</li>
<li>The Game Programing Wiki's <a href="http://content.gpwiki.org/index.php/OpenGL:Tutorials:Using_Quaternions_to_represent_rotation">Quaternion tutorial</a></li>
<li>Ogre3D's <a href="http://www.ogre3d.org/tikiwiki/Quaternion+and+Rotation+Primer">FAQ on quaternions</a>. Most of the 2nd part is ogre-specific, though.</li>
<li>Ogre3D's <a href="https://bitbucket.org/sinbad/ogre/src/3cbd67467fab3fef44d1b32bc42ccf4fb1ccfdd0/OgreMain/include/OgreVector3.h?at=default">Vector3D.h</a> and <a href="https://bitbucket.org/sinbad/ogre/src/3cbd67467fab3fef44d1b32bc42ccf4fb1ccfdd0/OgreMain/src/OgreQuaternion.cpp?at=default">Quaternion.cpp</a></li><br />
</ol></p>
<h1>Cheat-sheet</h1><br />
How do I know it two quaternions are similar ?</p>
<p>When using vector, the dot product gives the cosine of the angle between these vectors. If this value is 1, then the vectors are in the same direction.</p>
<p>With quaternions, it's exactly the same:</p>
<pre class="brush:cpp">float matching =&nbsp;quaternion::dot(q1,&nbsp;q2);<br />
if ( abs(matching-1.0) < 0.001 ){<br />
    // q1 and q2 are similar<br />
}</pre><br />
You can also get the angle between q1 and q2 by taking the acos() of this dot product.</p>
<h2>How do I apply a rotation to a point ?</h2><br />
You can do the following:</p>
<pre class="brush:cpp">rotated_point = orientation_quaternion *&nbsp; point;</pre><br />
... but if you want to compute your Model Matrix, you should probably convert it to a matrix instead.</p>
<p>Note that the center of rotation is always the origin. If you want to rotate around another point:</p>
<pre class="brush:cpp">rotated_point = origin + (orientation_quaternion * (point-origin));</pre></p>
<h2>How do I interpolate between 2 quaternions ?</h2><br />
This is called a SLERP: Spherical Linear intERPolation. With GLM, you can do this with mix:</p>
<pre class="brush:cpp">glm::quat interpolatedquat = quaternion::mix(quat1, quat2, 0.5f); // or whatever factor</pre></p>
<h2>How do I cumulate 2 rotations ?</h2><br />
Simple ! Just multiply the two quaternions together. The order is the same as for matrices, i.e. reverse:</p>
<pre class="brush:cpp">quat combined_rotation = second_rotation * first_rotation;</pre></p>
<h2>How do I find the rotation between 2 vectors ?</h2><br />
(in other words: the quaternion needed to rotate v1 so that it matches v2)</p>
<p>The basic idea is straightforward:</p>
<ul>
<li>The angle between the vectors is simple to find: the dot product gives its cosine.</li>
<li>The needed axis is also simple to find: it's the cross product of the two vectors.</li><br />
</ul><br />
The following algorithm does exactly this, but also handles a number of special cases:</p>
<pre class="brush:cpp">quat&nbsp;RotationBetweenVectors(vec3&nbsp;start,&nbsp;vec3&nbsp;dest){<br />
	start&nbsp;=&nbsp;normalize(start);<br />
	dest&nbsp;=&nbsp;normalize(dest);</p>
<p>	float&nbsp;cosTheta&nbsp;=&nbsp;dot(start,&nbsp;dest);<br />
	vec3&nbsp;rotationAxis;</p>
<p>	if&nbsp;(cosTheta&nbsp;<&nbsp;-1&nbsp;+&nbsp;0.001f){<br />
		//&nbsp;special&nbsp;case&nbsp;when&nbsp;vectors&nbsp;in&nbsp;opposite&nbsp;directions:<br />
		//&nbsp;there&nbsp;is&nbsp;no&nbsp;"ideal"&nbsp;rotation&nbsp;axis<br />
		//&nbsp;So&nbsp;guess&nbsp;one;&nbsp;any&nbsp;will&nbsp;do&nbsp;as&nbsp;long&nbsp;as&nbsp;it's&nbsp;perpendicular&nbsp;to&nbsp;start<br />
		rotationAxis&nbsp;=&nbsp;cross(vec3(0.0f,&nbsp;0.0f,&nbsp;1.0f),&nbsp;start);<br />
		if&nbsp;(gtx::norm::length2(rotationAxis)&nbsp;<&nbsp;0.01&nbsp;)&nbsp;//&nbsp;bad&nbsp;luck,&nbsp;they&nbsp;were&nbsp;parallel,&nbsp;try&nbsp;again!<br />
			rotationAxis&nbsp;=&nbsp;cross(vec3(1.0f,&nbsp;0.0f,&nbsp;0.0f),&nbsp;start);</p>
<p>		rotationAxis&nbsp;=&nbsp;normalize(rotationAxis);<br />
		return&nbsp;gtx::quaternion::angleAxis(180.0f,&nbsp;rotationAxis);<br />
	}</p>
<p>	rotationAxis&nbsp;=&nbsp;cross(start,&nbsp;dest);</p>
<p>	float&nbsp;s&nbsp;=&nbsp;sqrt(&nbsp;(1+cosTheta)*2&nbsp;);<br />
	float&nbsp;invs&nbsp;=&nbsp;1&nbsp;/&nbsp;s;</p>
<p>	return&nbsp;quat(<br />
		s&nbsp;*&nbsp;0.5f,&nbsp;<br />
		rotationAxis.x&nbsp;*&nbsp;invs,<br />
		rotationAxis.y&nbsp;*&nbsp;invs,<br />
		rotationAxis.z&nbsp;*&nbsp;invs<br />
	);</p>
<p>}</pre><br />
(You can find this function in common/quaternion_utils.cpp)</p>
<h2>I need an equivalent of gluLookAt. How do I orient an object towards a point ?</h2><br />
Use&nbsp;RotationBetweenVectors !</p>
<pre class="brush:cpp">//&nbsp;Find&nbsp;the&nbsp;rotation&nbsp;between&nbsp;the&nbsp;front&nbsp;of&nbsp;the&nbsp;object&nbsp;(that&nbsp;we&nbsp;assume&nbsp;towards&nbsp;+Z,<br />
//&nbsp;but&nbsp;this&nbsp;depends&nbsp;on&nbsp;your&nbsp;model)&nbsp;and&nbsp;the&nbsp;desired&nbsp;direction<br />
quat&nbsp;rot1&nbsp;=&nbsp;RotationBetweenVectors(vec3(0.0f,&nbsp;0.0f,&nbsp;1.0f),&nbsp;direction);</pre><br />
Now, you might also want to force your object to be upright:</p>
<pre class="brush:cpp">// Recompute desiredUp so that it's perpendicular to the direction<br />
// You can skip that part if you really want to force desiredUp<br />
vec3 right = cross(direction, desiredUp);<br />
desiredUp = cross(right, direction);</p>
<p>// Because of the 1rst rotation, the up is probably completely screwed up.<br />
// Find the rotation between the "up" of the rotated object, and the desired up<br />
vec3 newUp = rot1 * vec3(0.0f, 1.0f, 0.0f);<br />
quat rot2 = RotationBetweenVectors(newUp, desiredUp);</pre><br />
Now, combine them:</p>
<pre class="brush:cpp">quat targetOrientation =&nbsp;rot2&nbsp;*&nbsp;rot1;&nbsp;//&nbsp;remember,&nbsp;in&nbsp;reverse&nbsp;order.</pre><br />
Beware, "direction" is, well, a direction, not the target position ! But you can compute the direction simply: targetPos - currentPos.</p>
<p>Once you have this target orientation, you will probably want to interpolate between startOrientation and targetOrientation.</p>
<p>(You can find this function in common/quaternion_utils.cpp)</p>
<h2>How do I use LookAt, but limit the rotation at a certain speed ?</h2><br />
The basic idea is to do a SLERP ( = use glm::mix ), but play with the interpolation value so that the angle is not bigger than the desired value:</p>
<pre class="brush:cpp">float&nbsp;mixFactor&nbsp;=&nbsp;maxAllowedAngle&nbsp;/ angleBetweenQuaternions;<br />
quat&nbsp;result =&nbsp;glm::gtc::quaternion::mix(q1,&nbsp;q2,&nbsp;mixFactor);</pre><br />
Here is a more complete implementation, which deals with many special cases. Note that it doesn't use mix() directly as an optimization.</p>
<pre class="brush:cpp">quat&nbsp;RotateTowards(quat&nbsp;q1,&nbsp;quat&nbsp;q2,&nbsp;float&nbsp;maxAngle){</p>
<p>	if(&nbsp;maxAngle&nbsp;<&nbsp;0.001f&nbsp;){<br />
		//&nbsp;No&nbsp;rotation&nbsp;allowed.&nbsp;Prevent&nbsp;dividing&nbsp;by&nbsp;0&nbsp;later.<br />
		return&nbsp;q1;<br />
	}</p>
<p>	float&nbsp;cosTheta&nbsp;=&nbsp;dot(q1,&nbsp;q2);</p>
<p>	//&nbsp;q1&nbsp;and&nbsp;q2&nbsp;are&nbsp;already&nbsp;equal.<br />
	//&nbsp;Force&nbsp;q2&nbsp;just&nbsp;to&nbsp;be&nbsp;sure<br />
	if(cosTheta&nbsp;>&nbsp;0.9999f){<br />
		return&nbsp;q2;<br />
	}</p>
<p>	// Avoid taking the long path around the sphere<br />
	if (cosTheta < 0){<br />
	&nbsp;&nbsp; &nbsp;q1 = q1*-1.0f;<br />
	&nbsp;&nbsp; &nbsp;cosTheta *= -1.0f;<br />
	}</p>
<p>	float&nbsp;angle&nbsp;=&nbsp;acos(cosTheta);</p>
<p>	//&nbsp;If&nbsp;there&nbsp;is&nbsp;only&nbsp;a&nbsp;2&deg;&nbsp;difference,&nbsp;and&nbsp;we&nbsp;are&nbsp;allowed&nbsp;5&deg;,<br />
	//&nbsp;then&nbsp;we&nbsp;arrived.<br />
	if&nbsp;(angle&nbsp;<&nbsp;maxAngle){<br />
		return&nbsp;q2;<br />
	}</p>
<p>	float&nbsp;fT&nbsp;=&nbsp;maxAngle&nbsp;/&nbsp;angle;<br />
	angle&nbsp;=&nbsp;maxAngle;</p>
<p>	quat&nbsp;res&nbsp;=&nbsp;(sin((1.0f&nbsp;-&nbsp;fT)&nbsp;*&nbsp;angle)&nbsp;*&nbsp;q1&nbsp;+&nbsp;sin(fT&nbsp;*&nbsp;angle)&nbsp;*&nbsp;q2)&nbsp;/&nbsp;sin(angle);<br />
	res&nbsp;=&nbsp;normalize(res);<br />
	return&nbsp;res;</p>
<p>}</pre><br />
You can use it like that:</p>
<pre class="brush:cpp">CurrentOrientation = RotateTowards(CurrentOrientation, TargetOrientation, 3.14f * deltaTime );</pre><br />
(You can find this function in common/quaternion_utils.cpp)</p>
<h2>How do I...</h2><br />
If you can't figure it out, drop us an email, and we'll add it to the list !</p>
