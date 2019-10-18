# Notes  #


+ Edit the build.sh script with updates on versions and download sites

+ Use the build.sh script to create the docker container

Login to docker.com

	docker login --username=XXXXX

Be sure to use the following command to tag the final image:

	docker tag 0e6f39bb7024 dream3d/dream3d:dream3d_sdk

where you need to find out the exact hash of the final image. And finally push the image to the repo

	docker push dream3d/dream3d


