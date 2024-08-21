---
title: now for the videos
tags: light project pendulumsim
math: true
---

so it turns out rendering 1000, or even 100, double pendulums takes a *lot* of time. it also turns out that youtube compression absolutely *sucks*.
<!--more-->
c++ has a bunch of different video processing libraries, the most well-known of which is probably [`ffmpeg`](https://ffmpeg.org/), but i actually decided to use [`opencv`](https://opencv.org/) -- it comes with `ffmpeg`, but its primary role is actually computer vision, so if i decide to work on anything of that sort i already know the basics of image/video processing. (granted, using c++ for computer vision might be more pain than it's worth...)

and that's how i got introduced to the absolutely *wonderful* world of video codecs. for what it's worth, you'd think a library like `opencv` would do a good job at specifying which codecs it comes pre-installed with and which ones it supports, but apparently not. i tried using H264/265, the most commonly used one, but apparently *no* -- `opencv` doesn't support the default windows codecs. i needed to install another library -- [`openh264`](https://github.com/cisco/openh264) -- just to create a damn mp4. 

and when the video did render, i severely understimated the amount of compression artifacts 5 million particles would create (i have 5000 particles per pendulum). alright, screw it -- i'm using `opencv`'s native `MJPG` codec. 

```c++
void saveFramebuffer(GLFWwindow* window) {

	static int fid = 0;
	static cv::VideoWriter outputVideo;
	static double start_time;
	//static int height, width;
	int const total_frame = video_length * fps * fps_factor;

	//int const total_frame = 120;
	if (fid == 0) {
		/*RECT WindowRect;
		GetWindowRect(hWnd, &WindowRect);
		width = WindowRect.right - WindowRect.left;
		height = WindowRect.bottom - WindowRect.top;*/
		outputVideo.open("video.avi",  /*Video Name*/
			cv::VideoWriter::fourcc('M', 'J', 'P', 'G'),                         /* fourcc */
			(double)fps,                      /*Yuchen: Frame Rate */
			cv::Size(window_size, window_size),  /*Yuchen: Frame Size of the Video  */
			true);                      /*Yuchen: Is Color                 */
		start_time = glfwGetTime();
	}
	else if (fid < total_frame) {
		cv::Mat pixels( /* num of rows */ window_size, /* num of cols */ window_size, CV_8UC3);
		glReadPixels(0, 0, window_size, window_size, GL_BGR, GL_UNSIGNED_BYTE, pixels.data);
		//cv::Mat cv_pixels( /* num of rows */ window_size, /* num of cols */ window_size, CV_8UC3);
		//for (int y = 0; y < window_size; y++) for (int x = 0; x < window_size; x++)
		//{
		//	cv_pixels.at<cv::Vec3b>(y, x)[2] = pixels.at<cv::Vec3b>(window_size - y - 1, x)[0];
		//	cv_pixels.at<cv::Vec3b>(y, x)[1] = pixels.at<cv::Vec3b>(window_size - y - 1, x)[1];
		//	cv_pixels.at<cv::Vec3b>(y, x)[0] = pixels.at<cv::Vec3b>(window_size - y - 1, x)[2];
		//}
		outputVideo << pixels;
		if (fid % 1000 == 0) {
			std::cout << fid << " out of " << total_frame << " frames generated.\tProgress: " << fid * 100.0 / total_frame << "%.\tEstimated time left: " << 1000 * ((glfwGetTime() - start_time) / fid) * (total_frame * 1.0 - fid) / (60 * 1000) << " minutes." << std::endl;
		}
	}
	else if (fid == total_frame) {
		std::cout << "Total rendering time : " << (glfwGetTime() - start_time) / 60 << " minutes." << std::endl;
		outputVideo.release();
		glfwSetWindowShouldClose(window, true);
	}
	fid++;
}
```

calling that every render loop creates a video.

oh yeah, one weird thing -- you know how images use RGB color values? apparently `opencv`'s images use the reverse, BGR values. so, at the start, when i specified `GL_RGB` in `glReadPixels()`, i'd get weird fals coloring that even invert colors couldn't fix. yeah, no, that's really weird. 

so here's the [playlist](https://youtube.com/playlist?list=PLFt0fKzoTd40GwZq71WrdJN3LV1i_AQfe) of all my pendulum simulations. what's really interesting is how chainging the initial position from 160 degrees to 170 degrees makes the situation so much more chaotic -- suddenly, all the pendulums swing *over* the top instead of their movement being sorta restricted to the lower half of the circle. it seems like there's some energy threshold these pendulums pass between 160 and 170 degrees, allowing them to circle back around. as you can see, youtube's compression is quite bad, and you can hardly make anything out. so, here's a [folder](https://terabox.com/s/1FOTD3N_K13zCadkiJYhHXA) full of actually viewable, non-480p video. 

--- 

the only problem is that i'm restricted by computing power; it turns out that solving thousands of diffeq at the same time places a heavy strain on my computer, and so does storing 5 million points' worth of particle data. so, in the future, i can precompute the positions and particle trails of pendulums and store that data elsewhere. this leaves the only work in the render loop to be reading data and drawing the read data. 

but honestly, i think i'm mostly done with this project for now; i could do other physics simulations (like 3-body gravity simulations or light beam simulations) but i'm going to find an easier-to-use graphics library for that, i think. using openGL for this project was mostly cause i wanted to learn it, not for anything else; i think i'd work faster with an easier-to-use library/framework. 

--- 

y'know, i might completely switch gears for my next project. i want to work on a tetris ai. [these things](https://www.youtube.com/watch?v=SfxtnHXO1QE). i actually played tetr.io until i got ss, so i kinda want to design my own tetris bot. 

only problem is, unlike this, i have no clue where to start. 

this should be fun~