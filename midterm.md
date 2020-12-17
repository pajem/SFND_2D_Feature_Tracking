# Camera-based 2D Feature Tracking Mid-Term Project

This file contains documentation for the mid-term project rubric points.

The specifications of the system used to build and run the program are:

- Intel® Core™ i7-8550U w/ integrated UHD 620 graphics and 16GB of RAM

- Ubuntu 20.04

- g++ 9.3

- OpenCV 4.5.1

## MP.1 Data Buffer Optimization

The ring buffer was implemented by...

- first removing elements in the front of `dataBuffer` until its size is 1 element less than the `dataBufferSize`.

- and then insert the new data at the end of `dataBuffer`.

## MP.2 Keypoint Detection

`HARRIS`, `FAST`, `BRISK`, `ORB`, `AKAZE`, `SIFT` keypoint detectors were added in addition to the already implemented `SHITOMASI` detector.

Other than `HARRIS`, all detectors were created using their corresponding factory functions (`create()`), and were stored in a pointer to their parent class. And then using the parent class interface `cv::Feature2D::detect()` function, the keypoint detection was executed. Note the during creation of the `detector`, the default constructor argument values were used, as they were assumed to already provide sufficient performance for this project.

On the other hand, `HARRIS` implementation needed more lines of code and was implemented in a separate function since there was no child class of `cv::Feature2D` for Harris detector. `cv::cornerHarris` was the function used to generate an image wherein the corners are highlighted, and then feature points were selected if the corner exceeded a threshold value. Non-maximum suppression was also implemented by selecting the keypoint with the highest response among all overlapping keypoints.

Results were then verified visually for each detector. Each detector showed reasonable detected keypoints.

Note that to make verification easier, the program was changed to accept the detector type as input argument when running the program.

## MP.3 Keypoint Removal

Removal of keypoints was straightforward given that the region-of-interest coordinates and dimensions are already provided and fixed. It was implemented with one function, `std::vector::erase()`, and combining it with `std::remove_if()`. This is known as the `erase-remove` idiom in C++. The `cv::Rect` class has a function for checking if a point is inside it or not. This function, `cv::Rect::contains()` is used as the condition in `std::remove_if()`.

Result were again verified visually. Since the `cv::Rect` position and size is same for all images, it was expected that some points that do not belong to the target vehicle in front are not be filtered-out.

## MP.4 Keypoint Descriptors

Similar to detectors, the different descriptor extractors were created from their corresponding factory functions, stored in a pointer to their parent class of type `cv::DescriptorExtractor`, and descriptor extraction was executed using the `cv::DescriptorExtractor::compute()` function. Default constructor argument values were use to create the descriptor extractors.

Again, to make verification easier, the program was changed to accept the descriptor extractor type as input argument when running the program.

## MP.5 Descriptor Matching

The `FLANN` descriptor matcher has the same parent class as the already-implemented `BF` matcher. It was also created using its factory function (`create()`).

One thing to consider was the `FLANN` matcher only accepted descriptor values of type `float`. But only the `SIFT` descriptor extractor generated `float` descriptor values, therefore it was necessary to add a check if the descriptor type was `float` and then convert to `float` if not.

Also, K-nearest neighbor selector with `k=2` was implemented. `k=2` means that the 2 best matches for a descriptor are kept. 

Matcher type and selector type was also made as input argument of the program.

## MP.6 Descriptor Distance Ratio

For `KNN` selector, descriptor distance ratio test was used to determine if the match will be discarded or not. If the ratio of the 2 best matches are within the treshold, the better match of the two is added to the set of matches.

## MP.7 Performance Evaluation 1

For the 10 images, the average number of detected keypoints was estimated using the program logs. The keypoints counted are after filtering for the keypoints that belong to the vehicle in the front.

| Detector Type | Number of Keypoints | Neighborhoods description                                                                            |
| ------------- | ------------------- | ---------------------------------------------------------------------------------------------------- |
| SHITOMASI     | 118                 | smallest neighnorhood size, distributed evenly                                                       |
| HARRIS        | 26                  | also small but bigger than SHITOMASI, distributed evenly but low density                             |
| FAST          | 410                 | same as HARRIS in terms of size, but really dense                                                    |
| BRISK         | 276                 | ranging from small to big sizes, lots of overlaps, and clustered along the outer edges of the object |
| ORB           | 106                 | biggest size, no small ones, and keypoints are not distributed well, some almost look concentric.    |
| AKAZE         | 167                 | mid size, noticably distributed along the outer edges of the object                                  |
| SIFT          | 139                 | ranging from small to big sizes, most keypoints are along the outer edges of the object              |

## MP.8 Performance Evaluation 2

Using `MAT_BF` and `SEL_KNN` w/ descriptor distance ration threshold `0.8`, the table below shows an estimated average number of matches across the 10 images.

 The first column contains the estimated average number of keypoints detected which serves as context to the number of matches. The second column lists the detectors and the top row lists the descriptor extractors.

| Number of Keypoints |           | BRISK | BRIEF | ORB | FREAK | AKAZE | SIFT |
| ------------------- | --------- | ----- | ----- | --- | ----- | ----- | ---- |
| 118                 | SHITOMASI | 85    | 105   | 100 | 85    | X     | 105  |
| 26                  | HARRIS    | 16    | 20    | 18  | 16    | X     | 18   |
| 410                 | FAST      | 240   | 310   | 310 | 250   | X     | 310  |
| 276                 | BRISK     | 170   | 190   | 170 | 170   | X     | 180  |
| 106                 | ORB       | 83    | 60    | 85  | 46    | X     | 85   |
| 167                 | AKAZE     | 135   | 140   | 130 | 130   | 140   | 140  |
| 139                 | SIFT      | 66    | 70    | XX  | 66    | X     | 170  |

> X - `AKAZE` descriptor extractor only works together with `AKAZE` keypoints.
>
> XX- `ORB` descriptor extractor does not work with `SIFT` keypoints. OpenCV gives `OutOfMemoryError`.

## MP.9 Performance Evaluation 3

The table below shows an estimated average execution time in `ms` of keypoint detection and descriptor extraction for each combination of detector and descriptor extractor.

The first column contains the estimated average number of keypoints detected which serves as context to the execution time. The second column lists the detectors and the top row lists the descriptor extractors.

| Number of Keypoints |           | BRISK | BRIEF | ORB | FREAK | AKAZE | SIFT |
| ------------------- | --------- | ----- | ----- | --- | ----- | ----- | ---- |
| 118                 | SHITOMASI | 17    | 16    | 18  | 42    | X     | 33   |
| 26                  | HARRIS    | 19    | 19    | 21  | 45    | X     | 30   |
| 410                 | FAST      | 8     | 4     | 7   | 30    | X     | 24   |
| 276                 | BRISK     | 40    | 38    | 47  | 64    | X     | 61   |
| 106                 | ORB       | 10    | 9     | 20  | 37    | X     | 37   |
| 167                 | AKAZE     | 60    | 59    | 66  | 87    | 100   | 76   |
| 139                 | SIFT      | 87    | 86    | XX  | 115   | X     | 155  |

## TOP3 Keypoint Detector and Descriptor Extractor Combinations

1. `FAST` detector, `BRIEF` descriptor extractor

    - This combination has the fastest processing time which is essential for a real-time system.

    - `FAST` resulted in the most keypoints detected.

    - In addition, compared to other descriptor extractors, `BRIEF` provided the most number of matches.

2. `BRISK` detector, `BRIEF` descriptor extractor

    - `BRISK` detector is not fast but number of detected keypoints is 2nd compared to `FAST`, and the bigger sizes should provide more robustness and accuracy when extracting and matching descriptors.

    - Among descriptor extractors paired with `FAST`, `BRIEF` was the fastest and at the same time resulted in the most number of matches.

3. `ORB` detector, `BRISK` descriptor extractor

    - Although low in quantity of detected keypoints, the relatively larger keypoints size of `ORB` should allow for the best robustness and accuracy when extracting and matching descriptors. Quality over quantity.

    - Also, the processing time is fast.