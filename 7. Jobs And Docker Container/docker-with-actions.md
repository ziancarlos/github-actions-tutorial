Docker Refreshments
- Dockerfile is to create an image

To Build an image from a dockerfile
- docker build -t custom-image-name -f Dockerfile.Name ./locations  

After creating an image from docker build we can create a container from the image we have buikld
docker create --name custom-container-name custom-image-name

then we can run container
docker create --name custom-container-name custom-image-name