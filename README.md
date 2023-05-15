# Installing and Running OpenPose Docker on SSRDE
## Setting Up OpenPose Environment
1. Log into SSRDE
2. Create directory where you will be running Openpose. In this example, the directory is called "openpose_testing" so to make the directory, enter the command `mkdir openpose_testing`
3. In the directory, enter the command:
`singularity pull docker://szreik/openpose-cpu-face:latest`
4. When completed, you should see a file named "openpose-cpu-face_latest.sif" in your directory. It can be renamed to whatever you like as long as it has .sif extension
5. In the same directory where you created the Docker image (file with .sif extension), create a new directory to store input videos. In this example, the directory is called "videodata" so enter the command `mkdir videodata`. Make sure you are in the openpose_testing directory when you enter this command.
6. Inside the "videodata" directory, make a directory named "json" by entering `mkdir json`.

## Uploading videos

You can upload videos one of two ways:
* On Mac, you can connect to the server through Finder so you can visually see the files and drag them from your local machine to SSRDE. [How to Connect to Remote Server on Mac](https://www.switchingtomac.com/tutorials/osx/connecting-to-a-remote-or-local-server/)

* Using the scp command:
    * On you local machine (not in SSRDE!) go to the directory where you have your video, and enter the command `scp {NAME OF VIDEO FILE} {SSRDE USERNAME}@ssrde.ucsd.edu:/{PATH TO STORE VIDEO ON SSRDE}`. Once it fully uploads you should be able to find the video in the SSRDE path you inputted.

    * Using the example path names we created above, and considering that I am storing my "openpose_testing" directory in /sphere/greene-lab/lab_members/salma/, the scp command would look like this:  `scp video.mp4 szreik@ssrde.ucsd.edu:/sphere/greene-lab/lab_members/salma/openpose_testing/videodata`

## Running OpenPose on Videos

1. Now that our video is on SSRDE, we can send a job to SSRDE to run Openpose! 

    To do this, you need a shell script. The current OpenPose script can be found in /sphere/greene-lab/lab_members/salma/openpose_testing and is named "run_openpose.sh". The command to run OpenPose from my directories is already there, but if you created these directories somewhere else you will just need to alter the command like so:
`singularity exec -B {NAME OF FULL PATH TO VIDEO DATA DIRECTORY}:/data {NAME OF DOCKER IMAGE FILE} \
/openpose/build/examples/openpose/openpose.bin --model_folder /openpose/models \
—number_people_max 1 --render_pose 0 --face --face_render 1 \
--video /data/$1.mp4 --display 0 \
--write_video /data/$1_openpose.mp4 --keypoint_scale 4 \
--write_json /data/json/$1`

2. To queue the job on SSRDE, you need to enter the following command: `sbatch -J {NAME OF JOB} -p general --mail-user={YOUR EMAIL} --mail-type=ALL -o {PATH TO OPENPOSE TESTING DIRECTORY}/JOB NAME}_output.o%j.txt {PATH TO RUN OPENPOSE SCRIPT}/run_openpose.sh {NAME OF VIDEO FILE WITHOUT .mp4 EXTENSION}`

    For me, the command will look like this if I am running it on a video named pt102.mp4: `sbatch -J PT102 -p general --mail-user=szreik@ucsd.edu --mail-type=ALL -o /sphere/greene-lab/lab_members/salma/openpose_testing/PT102_output.o%j.txt /sphere/greene-lab/lab_members/salma/openpose_testing/run_openpose.sh pt102

3. You shoud now be able to enter the command `squeue` and see your job in the queue. You will get an email when the job starts/ends and you can find the output of running the script in your openpose_testing directory
