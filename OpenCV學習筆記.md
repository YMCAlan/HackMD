# Opencv C++ 學習筆記

## `cv::Ptr`
C11為了要解決記憶體洩漏問題(Memory Leak)的問題，便引進了「Smart Pointer」
`cv::Ptr`為OpenCV的Smart pointer Template

- 自動回收機制 : 使用完不用特別`delete`

## Data Struct
```cpp
struct CV_EXPORTS_W_SIMPLE ImageFeatures
{
    CV_PROP_RW int img_idx;
    CV_PROP_RW Size img_size;
    CV_PROP_RW std::vector<KeyPoint> keypoints;
    CV_PROP_RW UMat descriptors;
    CV_WRAP std::vector<KeyPoint> getKeypoints() { return keypoints; }
};
```

```cpp
CV_EXPORTS_W void computeImageFeatures(
    const Ptr<Feature2D> &featuresFinder,
    InputArrayOfArrays  images,
    CV_OUT std::vector<ImageFeatures> &features,
    InputArrayOfArrays masks = noArray());
```
- **featuresFinder**  
Abstract base class for 2D image feature detectors and descriptor extractors
- **images**  
Input Images
- **features**  
Structure containing image keypoints and descriptors.
- 
```cpp
struct CV_EXPORTS CameraParams
{
    CameraParams();
    CameraParams(const CameraParams& other);
    const CameraParams& operator =(const CameraParams& other);
    Mat K() const;

    double focal; // Focal length
    double aspect; // Aspect ratio
    double ppx; // Principal point X
    double ppy; // Principal point Y
    Mat R; // Rotation
    Mat t; // Translation
};
```
- **K matrix**  
camera's intrinsic matrix, include Focal length and Principle point
- **Focal** 
Focal length
- **Aspect**  
Scale factor
- **ppx and ppy**  
Principle point
- **R matrix**  
Rotation matrix
- **T matrix**
Translation matrix
```cpp
struct CV_EXPORTS_W_SIMPLE MatchesInfo
{
    MatchesInfo();
    MatchesInfo(const MatchesInfo &other);
    MatchesInfo& operator =(const MatchesInfo &other);

    CV_PROP_RW int src_img_idx;
    CV_PROP_RW int dst_img_idx;       //!< Images indices (optional)
    CV_PROP_RW std::vector<DMatch> matches;
    CV_PROP_RW std::vector<uchar> inliers_mask;    //!< Geometrically consistent matches mask
    CV_PROP_RW int num_inliers;                    //!< Number of geometrically consistent matches
    CV_PROP_RW Mat H;                              //!< Estimated transformation
    CV_PROP_RW double confidence;                  //!< Confidence two images are from the same panorama
    CV_WRAP std::vector<DMatch> getMatches() { return matches; }
    CV_WRAP std::vector<uchar> getInliers() { return inliers_mask; }
};
```



