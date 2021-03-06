---
layout: page
status: publish
published: true
title: 'Tutorial 14 : Render To Texture'
date: '2011-05-26 19:33:15 +0200'
date_gmt: '2011-05-26 19:33:15 +0200'
categories: [tuto]
order: 60
tags: []
language: cn
---
<p>Render-To-Texture is a handful method to create a variety of effects. The basic idea is that you render a scene just like you usually do, but this time in a texture that you can reuse later.</p>
<p>Applications include in-game cameras, post-processing, and as many GFX as you can imagine.</p>
<h1>Render To Texture</h1><br />
We have three tasks : creating the texture in which we're going to render ; actually rendering something in it ; and using the generated texture.</p>
<h2>Creating the Render Target</h2><br />
What we're going to render to is called a Framebuffer. It's a container for textures and an optional depth buffer. It's created just like any other object in OpenGL :</p>
<pre class="brush: cpp">// The framebuffer, which regroups 0, 1, or more textures, and 0 or 1 depth buffer.<br />
GLuint FramebufferName = 0;<br />
glGenFramebuffers(1, &amp;FramebufferName);<br />
glBindFramebuffer(GL_FRAMEBUFFER, FramebufferName);</pre><br />
Now we need to create the texture which will contain the RGB output of our shader. This code is very classic :</p>
<pre class="brush: cpp">// The texture we're going to render to<br />
GLuint renderedTexture;<br />
glGenTextures(1, &amp;renderedTexture);</p>
<p>// "Bind" the newly created texture : all future texture functions will modify this texture<br />
glBindTexture(GL_TEXTURE_2D, renderedTexture);</p>
<p>// Give an empty image to OpenGL ( the last "0" )<br />
glTexImage2D(GL_TEXTURE_2D, 0,GL_RGB, 1024, 768, 0,GL_RGB, GL_UNSIGNED_BYTE, 0);</p>
<p>// Poor filtering. Needed !<br />
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);<br />
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);</pre><br />
We also need a depth buffer. This is optional, depending on what you actually need to draw in your texture; but since we're going to render Suzanne, we need depth-testing.</p>
<pre class="brush: cpp">// The depth buffer<br />
GLuint depthrenderbuffer;<br />
glGenRenderbuffers(1, &amp;depthrenderbuffer);<br />
glBindRenderbuffer(GL_RENDERBUFFER, depthrenderbuffer);<br />
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT, 1024, 768);<br />
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_RENDERBUFFER, depthrenderbuffer);</pre><br />
Finally, we configure our framebuffer</p>
<pre class="brush: cpp">// Set "renderedTexture" as our colour attachement #0<br />
glFramebufferTexture(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, renderedTexture, 0);</p>
<p>// Set the list of draw buffers.<br />
GLenum DrawBuffers[1] = {GL_COLOR_ATTACHMENT0};<br />
glDrawBuffers(1, DrawBuffers); // "1" is the size of DrawBuffers</pre><br />
Something may have gone wrong during the process, depending on the capabilities of the GPU. This is how you check it :</p>
<pre class="brush: cpp">// Always check that our framebuffer is ok<br />
if(glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)<br />
return false;</pre></p>
<h2>Rendering to the texture</h2><br />
Rendering to the texture is straightforward. Simply bind your framebuffer, and draw your scene as usual. Easy !</p>
<pre class="brush: cpp">// Render to our framebuffer<br />
glBindFramebuffer(GL_FRAMEBUFFER, FramebufferName);<br />
glViewport(0,0,1024,768); // Render on the whole framebuffer, complete from the lower left corner to the upper right</pre><br />
The fragment shader just needs a minor adaptation :</p>
<pre class="brush: cpp">layout(location = 0) out vec3 color;</pre><br />
This means that when writing in the variable "color", we will actually write in the Render Target 0, which happens to be our texure because&nbsp;DrawBuffers[0] is&nbsp;GL_COLOR_ATTACHMENT<em>i</em>, which is, in our case,&nbsp;<em>renderedTexture</em>.</p>
<p>To recap :</p>
<ul>
<li><em>color</em> will be written to the first buffer because of layout(location=0).</li>
<li>The first buffer is&nbsp;&nbsp;GL_COLOR_ATTACHMENT0&nbsp;because of&nbsp;DrawBuffers[1] = {GL_COLOR_ATTACHMENT0}</li>
<li>GL_COLOR_ATTACHMENT0 has <em>renderedTexture</em> attached, so this is where your color is written.</li>
<li>In other words, you can replace&nbsp;GL_COLOR_ATTACHMENT0 by&nbsp;GL_COLOR_ATTACHMENT2 and it will still work.</li><br />
</ul><br />
Note : there is no layout(location=i) in OpenGL < 3.3, but you use glFragData[i] = mvvalue anyway.</p>
<div><span style="font-size: medium;"><span style="line-height: 24px;"><br />
</span></span></div></p>
<h2>Using the rendered texture</h2><br />
We're going to draw a simple quad that fills the screen. We need the usual buffers, shaders, IDs, ...</p>
<pre class="brush: cpp">// The fullscreen quad's FBO<br />
GLuint quad_VertexArrayID;<br />
glGenVertexArrays(1, &amp;quad_VertexArrayID);<br />
glBindVertexArray(quad_VertexArrayID);</p>
<p>static const GLfloat g_quad_vertex_buffer_data[] = {<br />
    -1.0f, -1.0f, 0.0f,<br />
    1.0f, -1.0f, 0.0f,<br />
    -1.0f,&nbsp; 1.0f, 0.0f,<br />
    -1.0f,&nbsp; 1.0f, 0.0f,<br />
    1.0f, -1.0f, 0.0f,<br />
    1.0f,&nbsp; 1.0f, 0.0f,<br />
};</p>
<p>GLuint quad_vertexbuffer;<br />
glGenBuffers(1, &amp;quad_vertexbuffer);<br />
glBindBuffer(GL_ARRAY_BUFFER, quad_vertexbuffer);<br />
glBufferData(GL_ARRAY_BUFFER, sizeof(g_quad_vertex_buffer_data), g_quad_vertex_buffer_data, GL_STATIC_DRAW);</p>
<p>// Create and compile our GLSL program from the shaders<br />
GLuint quad_programID = LoadShaders( "Passthrough.vertexshader", "SimpleTexture.fragmentshader" );<br />
GLuint texID = glGetUniformLocation(quad_programID, "renderedTexture");<br />
GLuint timeID = glGetUniformLocation(quad_programID, "time");</pre><br />
Now you want to render to the screen. This is done by using 0 as the second parameter of glBindFramebuffer.</p>
<pre class="brush: cpp">// Render to the screen<br />
glBindFramebuffer(GL_FRAMEBUFFER, 0);<br />
glViewport(0,0,1024,768); // Render on the whole framebuffer, complete from the lower left corner to the upper right</pre><br />
We can draw our full-screen quad with such a shader:</p>
<pre class="brush:fs">#version 330 core</p>
<p>in vec2 UV;</p>
<p>out vec3 color;</p>
<p>uniform sampler2D renderedTexture;<br />
uniform float time;</p>
<p>void main(){<br />
    color = texture( renderedTexture, UV + 0.005*vec2( sin(time+1024.0*UV.x),cos(time+768.0*UV.y)) ).xyz;<br />
}</pre><br />
&nbsp;</p>
<p>This code simply sample the texture, but adds a tiny offset which depends on time.</p>
<h1>Results</h1><br />
&nbsp;</p>
<p><a href="http://www.opengl-tutorial.org/wp-content/uploads/2011/05/wavvy.png"><img class="alignnone size-large wp-image-326" title="wavvy" src="http://www.opengl-tutorial.org/wp-content/uploads/2011/05/wavvy-1024x793.png" alt="" width="640" height="495" /></a></p>
<h1>Going further</h1></p>
<h2>Using the depth</h2><br />
In some cases you might need the depth when using the rendered texture. In this case, simply render to a texture created as follows :</p>
<pre class="brush: cpp">glTexImage2D(GL_TEXTURE_2D, 0,GL_DEPTH_COMPONENT24, 1024, 768, 0,GL_DEPTH_COMPONENT, GL_FLOAT, 0);</pre><br />
("24" is the precision, in bits. You can choose between 16, 24 and 32, depending on your needs. Usually 24 is fine)</p>
<p>This should be enough to get you started, but the provided source code implements this too.</p>
<p>Note that this should be somewhat slower, because the driver won't be able to use some optimisations such as <a href="http://fr.slideshare.net/pjcozzi/z-buffer-optimizations">Hi-Z</a>.</p>
<p>In this screenshot, the depth levels are artificially "prettified". Usually, its much more difficult to see anything on a depth texture. Near = Z near 0 = black, far = Z near 1 = white.</p>
<p><a href="http://www.opengl-tutorial.org/wp-content/uploads/2011/05/wavvydepth.png"><img class="alignnone size-large wp-image-337" title="wavvydepth" src="http://www.opengl-tutorial.org/wp-content/uploads/2011/05/wavvydepth-1024x793.png" alt="" width="640" height="495" /></a></p>
<h2>Multisampling</h2><br />
You can write to multisampled textures instead of "basic" textures : you just have to replace glTexImage2D by <a href="http://www.opengl.org/sdk/docs/man3/xhtml/glTexImage2DMultisample.xml">glTexImage2DMultisample</a> in the C++ code, and sampler2D/texture by sampler2DMS/texelFetch in the fragment shader.</p>
<p>There is a big caveat, though : texelFetch needs another argument, which is the number of the sample to fetch. In other words, there is no automatic "filtering" (the correct term, when talking about multisampling, is "resolution").</p>
<p>So you may have to resolve the MS texture yourself, in another, non-MS texture, thanks to yet another shader.</p>
<p>Nothing difficult, but it's just bulky.</p>
<h2>Multiple Render Targets</h2><br />
You may write to several textures at the same time.</p>
<p>Simply create several textures (all with the correct and same size !), call glFramebufferTexture with a different color attachement for each, call glDrawBuffers with updated parameters ( something like (2,{GL_COLOR_ATTACHMENT0,GL_COLOR_ATTACHMENT1}})), and add another output variable in your fragment shader :</p>
<pre class="brush:fs">layout(location = 1) out vec3 normal_tangentspace; // or whatever</pre><br />
Hint : If you effectively need to output a vector in a texture, floating-point textures exist, with 16 or 32 bit precision instead of 8... See <a href="http://www.opengl.org/sdk/docs/man/xhtml/glTexImage2D.xml">glTexImage2D</a>'s reference (search for GL_FLOAT).</p>
<p>Hint2 : For previous versions of OpenGL, use&nbsp;glFragData[1] = myvalue instead.</p>
<h1>Exercices</h1></p>
<ul>
<li>Try using glViewport(0,0,512,768); instead of glViewport(0,0,1024,768); (try with both the framebuffer and the screen)</li>
<li>Experiment with other UV coordinates in the last fragment shader</li>
<li>Transform the quad with a real transformation matrix. First hardcode it, and then try to use the functions of controls.hpp ; what do you notice ?</li><br />
</ul></p>
