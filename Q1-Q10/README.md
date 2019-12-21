## 前置问题：

[VS2019 配置Opencv](https://love1994love.github.io/2019/12/19/VS2019配置Opencv4)

[Vscode配置虚拟环境](https://code.visualstudio.com/docs/python/environments)

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

`vscode`中出现`cv2`报错解决办法：

[解决vscode+python+opencv3报错: Module 'cv2' has no 'imshow' member 下划波浪线报红](https://blog.csdn.net/y2c58s43d69g8h7G_g/article/details/96990126)

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

### `python`实现

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

### `C++实现`

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

