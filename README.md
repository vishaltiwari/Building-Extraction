## About
The objective of this project is to devise a method which can be used to extract buildings from an aerial image. Although more effective methods which use supervised classification and neural networks exist, our main aim is to use only image processing for this task.

## Method 1
![Method1](https://i.imgur.com/V7z32ll.png)

- In the first method we started off by applying segmentation techniques to the input aerial image to identify similar regions so that the processing for later on becomes easier. We tried two different segmentation techniques namely K-means segmentation method and Mean-shift segmentation method.

**K-means Segmentation:**
First we converted the given image to the LAB color space and then we classified the A & B namespace using k-means clustering. we chose the number of clusters to be four and got the result as shown in the figure ’b’ above. This happened because we are only considering the color space and not spatial properties and hence our result is not what was desired.

**Mean-shift Segmentation:**
For each data point, mean shift defines a window around it and computes the mean of data point. Then it shifts the center of window to the mean and repeats the algorithm till it converges. We are considering three things while calculating the mean-shift : the size of the kernel, the range bandwidth and the minimum number of pixels to be considered for a cluster.

![Segmentation](https://i.imgur.com/L9PWAx0.png)

- The next step was to identify the edges of the building. For this we tried two edge detection techniques - Canny edge detection and Sobel edge detection. Although the result of Sobel edge detection method is giving a clearer boundary of the buildings, we chose to go forward with the output given by the Canny edge detection technique. This was done because in the former method the output has a lot of noise and unwanted polygons whereas the latter is giving us a clearer idea where the buildings are which is essential for our next steps.

![Edge detection](https://i.imgur.com/56Mka2u.png)

- Next we applied Morphological operations - dilation followed by erosion(closing), then bridging the gaps between edges and finally shrinking and cleaning the image which gave us two sets of images - one having closed polygons and the others a set of non-closed edges.

![Morphological Operation](https://i.imgur.com/2vhg3wC.png)

- Our next step is to apply the convex hull method to the non-closed polygons edges to obtain polygons which identify where each building lie. Then we applied the vegetation and shadow masking(as explained in the next section) to remove the unwanted regions in the image.Using the output after masking and combining with the closed polygon image we applied local edge detection technique in the original image for those regions only.


## Method 2
The aim of this method is to mask out everything which is not building. Here we take an assumption that the area in the aerial images have only vegetation, shadows and roads. The rest is either building or a unclassifed area.

![Method2](https://i.imgur.com/FlhpP34.png)

- The first step of the algorithm is to mask out the vegetation from the aerial images. As we are only using the red, green and blue feature space and not Infrared space, we can’t find out NDVI value map and further mask out the vegetation. So to find out the vegetation area, we use the Visible Atmospherically Resistant Index (VARI) and RI index to find out the vegetation area.
The are defined as:
VARI = (g-r)/(g+r-b)
RI = g/b

![Veg Mask](https://i.imgur.com/izdkbek.png)

As this image has a lot of salt and pepper noise and blobs, so we applied a median filter and then an opening operations to get rid of the noise.

![Final Mask](https://i.imgur.com/bZjmtmm.png)

- The next step is to mask out the shadows from the image. To achieve our goal use the following shadow index to find the shadow region:
SI = (b-g) / (b+g)

And get the result shown in figure below. But it also contains a lot of salt and pepper noise, and also blobs, we use median filter followed by opening operations to find out the shadow mask.

![Shadow Mask](https://i.imgur.com/49SQpKt.png)

- Now after we have the vegetation and the shadow mask, we subtract the mask from the input image in all the bands and obtain an image which is free of vegetation and shadow. After masking out the image, we do a mean-shift segmentation and we get the the output as shown in the image below:

![Mask removal](https://i.imgur.com/mpxEVgF.png)

- The black area shown in the image is the masked out region. Now what we need to do is remove the road area. For this, first we take use of the fact that, in an area image, the roads will be connected to each other and thus the number of pixels belonging to the road will be maximum and all will be connected.

To reach our above aim, we first find the area which is not masked out, thus we subtract the masked image of vegetation and shadow. Then we further apply a closing operation to separate out the different components of the image. The output is shown below:

![Post Proc](https://i.imgur.com/QajdBrc.png)

- After obtaining this image, we find the skeleton of the image and we get the result as shown below.

![Skeleton](https://i.imgur.com/4J2mIcG.png)

- We can see that the line segments are broken and separated. To join them, we do an opening operation, and later find the connected components of the segments. After finding the connected components, we threshold all the segments on the basis of number of pixels in each segment. We retain only those segment which are above a specific threshold, which helps us decide with are roads and which are not. The results are shown below:

![Morph Skel](https://i.imgur.com/tayRYAM.png)

- After obtaining the skeleton of the road we do a region growth operation, and limit it to a specific limitation from the origin masked out image. After this we get the road mask as shown below.

![Road Mask](https://i.imgur.com/WFMHR18.png)

- As the final step we mask out the above area and from the segmented image and get the final result as shown below. The area shown in grey is the area that is classified as buildings.

![Final Result](https://i.imgur.com/SwuuSFJ.png)
