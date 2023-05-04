Download Link: https://assignmentchef.com/product/solved-csci5561-homework-5-stereo-reconstruction
<br>
In this assignment, you will implement a stereo reconstruction algorithm given two view images

Figure 1: In this assignment, you will implement a stereo reconstruction algorithm given two images.

You can download the skeletal code and data (left.bmp and right.bmp) from here: <a href="https://www-users.cs.umn.edu/~hspark/csci5561/HW5.zip">https://www-users.cs.umn.edu/</a><a href="https://www-users.cs.umn.edu/~hspark/csci5561/HW5.zip">~</a><a href="https://www-users.cs.umn.edu/~hspark/csci5561/HW5.zip">hspark/csci5561/HW5.zip</a>

You will fill main_stereo.m that takes input images and intrinsic parameters <strong>K</strong>, and produces a stereo disparity map.

<h1>1             SIFT Feature Matching</h1>

Figure 2: You will match points between I<sub>1 </sub>and I<sub>2 </sub>using SIFT features.

You will use VLFeat SIFT to extract keypoints and match between two views using k-nearest neighbor search. The matches will be filtered using the ratio test and bidirectional consistency check.

function [x1, x2] = FindMatch(I1, I2)

<strong>Input: </strong>two input gray-scale images with uint8 format.

<strong>Output: </strong>x1 and x2 are <em>n </em>× 2 matrices that specify the correspondence.

<strong>Description: </strong>Each row of x1 and x2 contains the (<em>x,y</em>) coordinate of the point correspondence in <em>I</em><sub>1 </sub>ad <em>I</em><sub>2</sub>, respectively, i.e., x1(i,:) &#x2194; x2(i,:). This matching function is similar to HW#2 except for bidirectional consistency check.

(Note) Except for SIFT extraction, you are not allowed to use VLFeat functions.

<h1>2             Fundamental Matrix Computation</h1>

Figure 3: Given matches, you will compute a fundamental matrix to draw epipolar lines.

function [F] = ComputeF(x1, x2)

<strong>Input: </strong>x1 and x2 are <em>n </em>× 2 matrices that specify the correspondence.

<strong>Output: </strong>F ∈ R<sup>3 </sup>is the fundamental matrix.

<strong>Description: </strong>F is robustly computed by the 8-point algorithm within RANSAC. Note that the rank of the fundamental matrix needs to be 2 (SVD clean-up should be applied.). You can verify the validity of fundamental matrix by visualizing epipolar line as shown in Figure 3.

(Note) Given the fundamental matrix, you can run the provided function:

[R1 C1 R2 C2 R3 C3 R4 C4] = ComputeCameraPose(F, K)

This function computes the four sets of camera poses given the fundamental matrix where R1 C1 ··· R4 C4 are rotation and camera center (represented in the world coordinate system) and K is the intrinsic parameter. These four configurations can be visualized in 3D as shown in Figure 4.

Figure 4: Four configurations of camera pose from a fundamental matrix.

<h1>3             Triangulation</h1>

Given camera pose and correspondences, you will triangulate to reconstruct 3D points.

function [X] = Triangulation(P1, P2, x1, x2)

<strong>Input: </strong>P1 and P2 are two camera projection matrices (R<sup>3×4</sup>). x1 and x2 are <em>n </em>× 2 matrices that specify the correspondence.

<strong>Output: </strong>X is <em>n </em>× 3 where each row specifies the 3D reconstructed point. <strong>Description: </strong>Use the triangulation method by linear solve, i.e.,

(Note) Use plot3 to visualize them as shown in Figure 5.

Figure 5: You can visualize four camera pose configurations with point cloud.

<h1>4             Pose Disambiguation</h1>

Given four configurations of relative camera pose and reconstructed points, you will find the best camera pose by verifying through 3D point triangulation.

function [R,C,X] = DisambiguatePose(R1,C1,X1,R2,C2,X2,R3,C3,X3,R4,C4,X4) <strong>Input: </strong>R1, C1, X1 ··· R4, C4, X4 are four sets of camera rotation, center, and 3D reconstructed points.

<strong>Output: </strong>R, C, X are the best camera rotation, center, and 3D reconstructed points. <strong>Description: </strong>The 3D point must lie in front of the both cameras, which can be tested by:

<strong>r </strong>0                                                         (1)

where <strong>r</strong><sub>3 </sub>is the 3<sup>rd </sup>row of the rotation matrix. In Figure 5, nValid means the number of points that are in front of both cameras. (b) configuration produces the maximum number of valid points, and therefore the best configuration is (b).

<h1>5             Stereo</h1>

(a) Rectified image 1                                                       (b) Rectified image 2

Figure 6: Stereo rectification.

Given the disambiguated camera pose, you will implement dense stereo matching between two views based on dense SIFT of VLFeat.

function [H1, H2] = ComputeRectification(K, R, C)

<strong>Input: </strong>The relative camera pose (R and C) and intrinsic parameter K.

<strong>Output: </strong>H1 and H2 are homographies that rectify the left and right images such that the epipoles are at infinity.

<strong>Description: </strong>Given the disambiguated camera pose, you can find the rectification rotation matrix, <strong>R</strong><sub>rect </sub>such that the x-axis of the images aligns with the baseline. Find the rectification homography <strong>H </strong>= <strong>KR</strong><sub>rect</sub><strong>R</strong><sup>T</sup><strong>K</strong><sup>−1 </sup>where <strong>R </strong>is the rotation matrix of the camera. The rectified images are shown in Figure 6. This rectification sends the epipoles to infinity where the epipolar line becomes horizontal.

(Note) You can use the provided image warping function im_w = WarpImage(im, H) to check your result.

Figure 7: Visualization of stereo match.

function [disp] = DenseMatch(im1, im2)

<strong>Input: </strong>two gray-scale rectified images with uint8 format.

<strong>Output: </strong>disparity map disp ∈ R<em><sup>H</sup></em><sup>×<em>W </em></sup>where <em>H </em>and <em>W </em>are the image height and width.

<strong>Description: </strong>Compute the dense matches across all pixels. Given a pixel, <strong>u </strong>in the left image, sweep along its epipolar line, <strong>l<sub>u</sub></strong>, and find the disparity, <em>d</em>, that produces the best match, i.e.,

<em>d </em>= argmin

where <strong>d</strong><sup>1</sup><strong><sub>u </sub></strong>is the dense SIFT descriptor at <strong>u </strong>on the left image and <strong>d</strong> is the SIFT descriptor at <strong>u</strong>+(<em>i,</em>0) (<em>i </em>pixel displaced along the x-axis) on the right image. Visualize the disparity map as shown in Figure 7.