
# Local Build # 

This failed ☹️

There were a bunch of issues with the `libopencv3.3.1` debians not being able to install properly. I tried installing these manually outside of the script, but that didn't improve things. This may be an issue because the computer currently has libopencv 4.2.0, which got installed with some of the ROS stuff I was working on. 

This made me wonder if this build script I was running was not really optimized for Ubuntu 20. I went into the file and was going to make changes before I realized (1) This would be more work than just changing a few lines of code, and (2) I should probably check the astrobee repo to see if anyone else already did this. 

# Local Build on the astrobee/develop branch #

This also failed 😥

This got a lot further along in the process, at least. There was a commit in the `develop` branch regarding "Ubuntu20 migration" (https://github.com/nasa/astrobee/pull/511) and this updated a lot of the build scripts in exactly the way I was looking for.

When building this, it didn't run into the same issues with libopencv and other packages, which was nice, but the build process stil failed at around 85-ish packages in. All of the other processes were abandoned after this. 

# Docker #

Since a lot of these issues seemed to be related to packages and dependencies, I figured I'd give the Docker method a shot. Installing Docker went smoothly, and the `docker/run.sh` script actually seemed to work 😀. Note: the full syntax for this command is `./run.sh --remote --args "rviz:=true sviz:=true"`

I also tried out the `./build.sh` file to locally build it with Docker, but this also failed multiple times (I was really hoping this would work 😥). Usually, what would happen is it would get to ~90/95 built, and then fail at `graph_localizer`. However, we probably don't need to worry about this too much if we can get the simulator/visualizer running without the full build process. 

Note that Docker Engine doesn't work for Ubuntu 16 (but this isn't really a big deal because the local build worked on the computer with 16)

I'm not totally sure if this running with Docker will make it more difficult to communicate between Gazebo/Bullet, but I can figure this out