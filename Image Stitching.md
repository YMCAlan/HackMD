# Image Stitching
目錄
[TOC]

## 簡介
影像拼貼就是將多張影像拼貼在一起，這多張影像會有彼此重疊的區域，因此我們可以將影像直接在有重疊地方進行貼圖，目的是讓使用者可以觀察到全景的影像，應用上會在車載影像、飛行器等等。

## 議題
:::warning
* 視差 
* 曝光差異
* 動態模糊
* 相機位置差異
* 相機為魚眼鏡頭(Fisheye Image)
* 相機為廣角鏡頭(Wide Angel Image)
* 影像融合的邊界處理
* 畸變參數的處理
:::

## OpenCV Image Stitching Process
### Parameters

- work_megapix
表示限制處理圖像的總像素數，當`work_megapix=0.6`表示處理0.6百萬像素質，素數不超過0.6百萬的大小，有效地降低運算量。
- compose_megapix
決定拼圖的時候，處理的總像素量該是多少
- seam_megapix
決定找縫線時圖像大小是多少，處理的總像素量該是多少

### Read Image and Resize
讀圖並根據需求縮放影像大小
應用流程有三個
1. 找特徵
2. 找縫線
3. 真正貼圖的時候

- 為什麼需要縮小影像
    - 在硬體限制情況下，處理影像大小不能太大，否則會破圖:cry:

\begin{align}
n &= h^{\prime}w^{\prime} \\
h^{\prime} &= h \times scale \\
w^{\prime} &= w \times scale \\
h^{\prime}w^{\prime} &= h w \times scale^2 \\ 
scale &= \sqrt{\frac{h^{\prime}w^{\prime}}{hw}} \\ &= \sqrt{\frac{n}{area}}
\end{align}

其中:
- $h^{\prime}, w^{\prime} :$ 縮放後影像大小
- $h, w :$ 原始影像大小
- $n :$ 表示 n 百萬像素質 
- $area :$ 原始影像的總像素值

### Feature Extraction 特徵提取
針對影像找特徵點，會給出特徵點的位置與描述子。
- 為什麼需要特徵提取?
    - 因為原始影像包含大量噪聲與不必要的細節
- 為什麼這些因素會是問題?
    - 在拼貼過程會增加時間雜度，並造成配準錯誤，也就是特徵匹配錯誤
- 為什麼需要找到對應的特徵點進行配準?
    - 拼貼影像一定有重疊部分，找到特徵點並進行匹配就是要找到哪些地方是重疊的，因為特徵點是包含了圖像空間關係與重疊物件區域訊息。
- 為什麼位置準確且重要?  
    - 人眼對於影像連續性很敏感，如果有配準錯誤的地方，很容易發現。

#### Method
- ORB
- SIFT
- AKAZE

### Feature Matching 特徵匹配
對於每一個特徵找到最符合的兩個特徵去match，並設計閥值過濾特徵距離，並且confidence要大於閥值
- 為什麼需要特徵匹配
    - 確保圖像是有相似的特徵點，並確保他們之間的對應關係
- 為甚麼需要自動找特徵點
    - 找尋所有特徵點在圖像間的關係是很耗費時間的，所以需要有方法能夠找到適合的點進行匹配

1. 進行特徵匹配
2. 檢查是否能夠找到transform
3. 建構點對點的對應關係
4. 找到pair-wise motion
   esitmate affine 
5. 找有多少個點在inlier
6. 將inlier的點重新建構一次homography的關係
*  `AffineBestOf2NearestMatcher`
*  `BestOf2NearestMatcher`
- [ ] `findHomography`理論 
- [ ] `RANSEC`方法

### Select Image 最大連通域連接
最大連通域我在猜是disjoint Sets
1. 建立併查集(Disjoint Sets)
2. 選擇分數過閥值的進行合併
3. 過濾掉不符合閥值的
* `leaveBiggestComponent`
- [ ] 併查集 Disjoint Sets

### Estimate Camera Parameters 估計相機的參數
是一個用於計算相機的內部參數，通常在相機校準和計算機視覺應用中使用。這些參數包括**焦距、光心、~~畸變係數~~**以及相機的**旋轉**和~~平移~~。
- 為什麼需要估計相機的參數?
    - 因為影像拼貼中需要多張影像無縫地貼在一起，因此需要確立每張影像間的空間關係進而實現多張影像的拼圖
- 估計相機的Focal Length
    - 藉由匹配的特徵的`homograph`去估
- 估計全局的運動Restore global motion
    - 最大展開樹
        - 建構graph，兩個圖像若有關係則添加邊連起來
        - 找到最大生成樹
        - 找到最大生成樹的中心
    - 廣度優先搜尋BFS
        - 對相同level的計算其CalcRotation
* `HomographyBasedEstimator`
* `AffineBasedEstimator`

[Theory behind estimating rotation](https://answers.opencv.org/question/40338/theory-behind-estimating-rotation-in-motion_estimatorscpp-calcrotation/)
### Refine Camera Parameters 校準相機參數
Camera Calibration and 3D Reconstruction
對先前估計得到的相機參數進行進一步的優化和調整，以提高影像拼貼的精度和品質。
* `BundleAdjusterReproj`
* `BundleAdjusterRay`
[光束法平拆](https://scm_mos.gitlab.io/vision/bundle-adjustment/)

### Wave Correct 波型校正
取得新的旋轉矩陣，能夠使圖像更加平滑


### Warp Image 影像扭曲
對圖像進行變換，包含旋轉、縮放、平移、透視。
```
1. 根據相機參數建立投影的map
2. 然後進行投影
3. 重新映射動作
```
- [ ] `Plane` 
將圖貼在某個平面上 

### Compensate Exposure 曝光補償
在拼接前的曝光補償，如果兩張影像差異太大會需要調整，
- 為什麼需要補償曝光？
   - 因為在影像拼貼中，不同圖像可能是在不同的光線條件下拍攝的，導致每張圖像的曝光程度不同。
- 為什麼不同圖像的曝光程度不同？
   - 因為光線的強度和方向可能因拍攝時的位置、時間或環境而有所不同。
- 為什麼不同曝光程度會導致問題？
   - 因為在拼貼時，不同曝光程度的圖像可能會造成色調和亮度的不一致，從而影響整體圖像的視覺效果。
- 為什麼色調和亮度的不一致會影響視覺效果？
   - 因為人眼對於色彩和亮度的感知是非常敏感的，即使是微小的差異也可能被視為不自然或者干擾。
- 為什麼需要保持視覺效果的一致性？
   - 因為影像拼貼的目的是創建一個連續且無縫的圖像，因此，需要補償曝光，確保不同圖像之間的色調和亮度一致，從而保持整體視覺效果的一致性和品質。

### Estimates Seams 估計縫線
找到最好的拼接的接縫

### Image Blending 影像融合
#### Method
- Feather-Blending
線性組合將影像融合非常簡單，依照下面公式
\begin{align}
I = \alpha I_{left} + (1 - \alpha)I_{right}
\end{align}
- Multi-Blending 



## Reference
[图像拼接现在还有研究的价值吗？有哪些可以研究的点？现在技术发展如何?](https://www.zhihu.com/question/34535199/answer/135169187)

[OpenCV教學](https://github.com/whdlgp/Stitching-tutorial-with-OpenCV/blob/master/get_camera_params/get_camera_params.md)

[相機內外參數](https://blog.techbridge.cc/2018/04/22/intro-to-pinhole-camera-model/)

[簡化版Code](https://zhuanlan.zhihu.com/p/71777362)

[c++opencv相機標定](https://zhuanlan.zhihu.com/p/607935061)

## To Do items
- [ ] Feature Extraction 演算法解說