## 前置问题：

[VS2019 配置Opencv](https://love1994love.github.io/2019/12/19/VS2019配置Opencv4)

[Vscode配置虚拟环境](https://code.visualstudio.com/docs/python/environments)

[`vscode`中出现`cv2`报错解决办法](https://blog.csdn.net/y2c58s43d69g8h7G_g/article/details/96990126)

## 问题一：通道交换

读取图像，然后将`text{RGB}`通道替换成`text{BGR}`通道，以便`Opencv`处理。

### `Python`实现

```python
import cv2

# function: BGR -> RGB
def BGR2RGB(img):
    b = img[:, :, 0].copy()
    g = img[:, :, 1].copy()
    r = img[:, :, 2].copy()

    # RGB > BGR
    img[:, :, 0] = r
    img[:, :, 1] = g
    img[:, :, 2] = b

    return img

# cv2.imread()的系数是按照 text{BGR}顺序排列的
img = cv2.imread("imori.jpg")

# BGR -> RGB，以便将图片保存
img = BGR2RGB(img)

# Save result
cv2.imwrite("out.jpg", img)
cv2.imshow("result", img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### `C++`实现

```C++
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <iostream>

//using cv::Mat;
using namespace cv;

// Channel swap : RGB --> BGR
Mat channel_swap(Mat img) {
	
	// get height and width 
	int width = img.cols;
	int height = img.rows;

	//prepare output : 8 bits unchar 3 channels
	Mat out = Mat::zeros(height, width, CV_8UC3);

	// each y, x 
	for (int y = 0; y < height; y++) {
		for (int x = 0; x < width; x++) {
			// R -> B
			out.at<Vec3b>(y, x)[0] = img.at<Vec3b>(y, x)[2];
			// B -> R
			out.at<Vec3b>(y, x)[2] = img.at<Vec3b>(y, x)[0];
			// G -> G
			out.at<Vec3b>(y, x)[1] = img.at<Vec3b>(y, x)[1];
		}
	}

	return out;
}
```

## 问题二：灰度化(Grayscale)

灰度是一种图像亮度的表示方式，计算公式为：
$$
Y = 0.2126\ R + 0.7152\ G + 0.0722\ B 
$$

### `Python`实现

```python
import cv2 
from cv2 import cv2
import numpy as np 

# Gray scale 
def BGR2GRAY(img):
    b = img[:, :, 0].copy()
    g = img[:, :, 1].copy()
    r = img[:, :, 2].copy()

    # Gray scale 
    out = 0.2126 * r + 0.7152 * g + 0.0722 * b
    out = out.astype(np.uint8)

    return out 

# Read image 
img = cv2.imread(r"C:\Users\Leowen\imori.jpg").astype(np.float)

# Grayscale 
out = BGR2GRAY(img)

# save result 
cv2.imwrite("out.jpg", out)
cv2.imshow("result", out)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### `C++`实现

```C++
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <iostream>

using namespace cv;

// BGR --> Gray
Mat BGR2GRAY(Mat img) {
	// get height and width 
	int height = img.rows;
	int width = img.cols;

	// prepare output 
	Mat out = Mat::zeros(height, width, CV_8UC1);

	// each y, x 
	for (int y = 0; y < height; y++) {
		for (int x = 0; x < width; x++) {
			// BGR -> GRAY
			out.at<uchar>(y, x) = 0.2126 * (float)img.at<Vec3b>(y, x)[2] \
				+ 0.7152 * (float)img.at<Vec3b>(y, x)[1] \
				+ 0.0722 * (float)img.at<Vec3b>(y, x)[0];
		}
	}

	return out;
}

int main(int argc, const char* argv[]) {
	// read image 
	Mat img = imread("imori.jpg", IMREAD_COLOR);

	// BGR -> Gray
	Mat out = BGR2GRAY(img);

	imwrite("out.jpg", out);
	imshow("sample", out);
	waitKey(0);
	destroyAllWindows();

	return 0;
}
```

## 问题三：二值化(Thresholding)

二值化是将图像使用黑和白两种颜色表示的方法。

我们将灰度的阈值设置为`128`来进行二值化，即： 
$$
y= \begin{cases} 0& (\text{if}\quad y < 128) \  & 255& (\text{else}) \end{cases}
$$

### `Python`实现

```python
import cv2 
from cv2 import cv2 
import numpy as np

# Gray scale 
def BGR2GRAY(img):
    b = img[:, :, 0].copy()
    g = img[:, :, 1].copy()
    r = img[:, :, 2].copy()

    # Gray scale 
    out = 0.2126 * r + 0.7152 * g + 0.0722 * b
    out.astype(np.uint8)

    return out 

# 二值化处理
def binarization(img, th=128):
    img[img < th] = 0
    img[img >= th] = 255
    return img 

# Read image 
img = cv2.imread("imori.jpg").astype(np.float32)

# RGB -> Grayscale
out = BGR2GRAY(img) 

# Grayscale -> binarization 
out = binarization(out)

# save result 
cv2.imwrite('out.jpg', out)
cv2.imshow('result', out)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### `C++`实现

```c++
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <iostream>
using namespace cv;

// BGR -> Grayscale 
Mat BGR2GRAY(Mat img) {
	// get height and width 
	int width = img.cols;
	int height = img.rows;

	// prepare output 
	Mat out = Mat::zeros(height, width, CV_8UC1);

	// each y, x 
	for (int y = 0; y < height; y++) {
		for (int x = 0; x < width; x++) {
			out.at<uchar>(y,x) = 0.2126 * (float)img.at<Vec3b>(y, x)[2] \
				+ 0.7152 * (float)img.at<Vec3b>(y, x)[1] \
				+ 0.0722 * (float)img.at<Vec3b>(y, x)[0];
		}
	}

	return out;
}

// Gray -> Binary 
Mat Binary(Mat gray, int th) {
	int width = gray.cols;
	int height = gray.rows; 

	// prepare output 
	Mat out = Mat::zeros(height, width, CV_8UC1);

	// each y, x 
	for (int y = 0; y < height; y++) {
		for (int x = 0; x < width; x++) {
			// Binary
			if (gray.at<uchar>(y, x) > th)
				out.at<uchar>(y, x) = 255;
			else
				out.at<uchar>(y, x) = 0;
		}
	}

	return out;
}

int main(void) {
	// read image 
	Mat img = imread("imori.jpg", IMREAD_COLOR);

	// BGR -> Gray
	Mat gray = BGR2GRAY(img);

	// Gray -> Binary
	Mat out = Binary(gray, 128);

	imshow("sample", out);
	waitKey(0);
	cvDestroyAllWindows();

	return 0;
}
```

## 问题四：大津二值化算法（Otsu's Method）

大津算法，也被称作最大类间方差法，是一种可以自动确定二值化中阈值的算法。

从**类内方差**和**类间方差**的比值计算得来：

- 小于阈值`t`的类记作`0`，大于阈值`t`的类记作`1`；
- `w_0`和`w_1`是被阈值`t`分开的两个类中的像素数占总像素数的比率（满足`w_0`+`w_1`=`1`）；
- `{S_0}^2`， `{S_1}^2`是这两个类中像素值的方差；
- `M_0`，`M_1`是这两个类的像素值的平均值；

即：

- 类内方差：
  $$
  {S_w}^2=w_0\ {S_0}^2+w_1\ {S_1}^2
  $$

- 类间方差：
  $$
  {S_b}^2 = w_0 \ (M_0 - M_t)^2 + w_1\ (M_1 - M_t)^2 = w_0\ w_1\ (M_0 - M_1) ^2
  $$

- 图像所有像素的方差：
  $$
  {S_t}^2 = {S_w}^2 + {S_b}^2 = \text{常数}
  $$

根据以上的式子，我们用以下的式子计算分离度`X`
$$
X = \frac{{S_b}^2}{{S_w}^2} = \frac{{S_b}^2}{{S_t}^2 - {S_b}^2} 
$$
也就是说： 
$$
\arg\max\limits_{t}\ X=\arg\max\limits_{t}\ {S_b}^2
$$
换言之，如果使
$$
{S_b}^2={w_0}\ {w_1}\ (M_0 - M_1)^2
$$
最大，就可以得到最好的二值化阈值`t`。

参考：

[大津算法（OTSU）](https://www.jianshu.com/p/ff7f9f00bd99)

![](D:\Git_love1994love\Image-processing-100\Q1-Q10\OTSU.png)

### `Python`实现

```python
import cv2
from cv2 import cv2
import numpy as np
def BGR2GRAY(img):
    b = img[:, :, 0].copy()
    g = img[:, :, 1].copy()
    r = img[:, :, 2].copy()

    # Gray scale
    out = 0.2126 * r + 0.7152 * g + 0.0722 * b
    out = out.astype(np.uint8)

    return out

# Otsu Binarization


def otsu_binarization(img):
    max_sigma = 0
    max_t = 0
    H, W = img.shape
    # determine threshold
    for _t in range(1, 256):
        # 小于阈值t
        v0 = img[np.where(img < _t)]
        m0 = np.mean(v0) if len(v0) > 0 else 0
        w0 = len(v0) / (H * W)
        # 大于阈值t
        v1 = img[np.where(img >= _t)]
        m1 = np.mean(v1) if len(v1) > 0 else 0
        w1 = len(v1) / (H * W)

        sigma_b_square = w0 * w1 * ((m0 - m1) ** 2)
        if sigma_b_square > max_sigma:
            max_sigma = sigma_b_square
            # 获得最好的二值化阈值t
            max_t = _t

    # Binarization
    print("Threshold >> {}".format(max_t))
    th = max_t 
    img [img < th] = 0
    img[img >= th] = 255 

    return img 

# Read image 
img = cv2.imread("imori.jpg").astype(np.float32)

# Grayscale
gray = BGR2GRAY(img)

# Otsu's binarization
out = otsu_binarization(gray)

# save result 
cv2.imwrite("out.jpg", out)
cv2.imshow("result", out)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### `C++`实现

```C++
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <iostream>
#include <math.h>

using namespace cv;
using namespace std;

// BGR -> Gray 
Mat BGR2GRAY(Mat img) {
	// get height and width
	int height = img.rows; 
	int width = img.cols;

	// prepare output 
	Mat out = Mat::zeros(height, width, CV_8UC1);

	// each y, x
	for (int y = 0; y < height; y++) {
		for (int x = 0; x < width; x++) {
			// BGR -> Gray
			out.at<uchar>(y,x) = 0.2126 * (float)img.at<cv::Vec3b>(y, x)[2] \
				+ 0.7152 * (float)img.at<cv::Vec3b>(y, x)[1] \
				+ 0.0722 * (float)img.at<cv::Vec3b>(y, x)[0];
		}
	}
	
	return out;
}

// Gray -> Binary
Mat Binarize_Otsu(Mat gray) {
	int width = gray.cols;
	int height = gray.rows;

	// determine threshold
	double w0 = 0, w1 = 0;
	double m0 = 0, m1 = 0;
	double max_sigma_b = 0, sigma_b_square = 0;
	int th = 0;
	int val;

	for (int t = 1; t < 256; t++) {

		w0 = 0, w1 = 0, m0 = 0, m1 = 0;

		for (int y = 0; y < height; y++) {
			for (int x = 0; x < width; x++) {
				val = (int)(gray.at<uchar>(y, x));

				if (val < t) {
					w0++;
					m0 += val;
				}
				else {
					w1++;
					m1 += val;
				}
			}
		}

		m0 /= w0;
		m1 /= w1;
		w0 /= (height * width);
		w1 /= (height * width);

		sigma_b_square = w0 * w1 * pow((m0 - m1), 2);

		if (sigma_b_square > max_sigma_b) {
			max_sigma_b = sigma_b_square;
			th = t;
		}
	}

	cout << "threshold >> " << th << endl;

	// prepare output 
	Mat out = Mat::zeros(height, width, CV_8UC1);

	// each y, x
	for (int y = 0; y < height; y++) {
		for (int x = 0; x < width; x++) {
			// Binarize
			if (gray.at<uchar>(y, x) > th)
				out.at<uchar>(y, x) = 255;
			else
				out.at<uchar>(y, x) = 0;
		}
	}

	return out;
}


// 主函数
int main(void) {

	// read image 
	Mat img = imread("imori.jpg", IMREAD_COLOR);

	// BGR -> Gray
	Mat gray = BGR2GRAY(img);

	// Gray -> Binary
	Mat out = Binarize_Otsu(gray);

	imshow("sample", out);
	waitKey(0);
	destroyAllWindows();

	return 0;
}
```

