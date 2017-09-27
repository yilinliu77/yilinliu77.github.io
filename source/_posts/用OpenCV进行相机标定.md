---
title: 用OpenCV进行相机标定
date: 2017-09-01 18:50:13
tags:
- Computer Vision

---
# 相机标定的原理

## 成像模型

相机在计算机视觉应用中起着重要作用，作为图像数据来源，影响着后续各个处理步骤。成像模型就是用数学公式刻画整个成像过程，即被拍摄物体空间点到照片成像点之间的几何变换关系。

总体上，相机成像可以分为四个步骤：刚体变换、透视投影、畸变校正和数字化图像。

![相机成像模型](http://otbwgn2nv.bkt.clouddn.com/34b5e49d0d138be3658b2159e7bb5a75.png)

### 刚体变换
刚体变换只改变物体的空间位置(平移)和朝向(旋转)，而不改变其形状，可用两个变量来描述：旋转矩阵R和平移向量t

![刚体变换](http://otbwgn2nv.bkt.clouddn.com/1ac9d44f769814d43792588d48c5ce3a.png)

齐次坐标下可写为：

![刚体变换](http://otbwgn2nv.bkt.clouddn.com/a38e43248e0052a0300c4e1952f815ad.png)

旋转矩阵R是正交矩阵，可通过罗德里格斯（Rodrigues）变换转换为只有三个独立变量的旋转向量：

![Rodrigues](http://otbwgn2nv.bkt.clouddn.com/7e4b581b5276314cf76a148e0be3832d.png)

因此，刚体变换可用6个参数来描述，这6个参数就称为相机的外参(Extrinsic)，相机外参决定了空间点从世界坐标系转换到相机坐标系的变换，也可以说外参描述了相机在世界坐标系中的位置和朝向。

### 透视投影
我们可以将透镜的成像简单地抽象成下图所示：

![透视投影](http://otbwgn2nv.bkt.clouddn.com/8e161a5c59f4dd7b03171e037a7fa475.png)

设 f=OB 表示透镜的焦距，m=OC 为像距，n=AO 为物距，有：

![焦距](http://otbwgn2nv.bkt.clouddn.com/2c2568cb708d6d0b0f51b9828ea8f7ee.png)

一般地，由于物距远大于焦距，即 n>>f，所以 m≈f，此时可以用小孔模型代替透镜成像：

![小孔成像](http://otbwgn2nv.bkt.clouddn.com/0f71e4f9c80acc9adc065707d2d3f775.png)

可得：

![](http://otbwgn2nv.bkt.clouddn.com/584e9339e90f767d3647d336babfe443.png)

齐次坐标下有：

![](http://otbwgn2nv.bkt.clouddn.com/26d8e6748631307ab47a3ab323c6f13f.png)

如果将成像平面移到相机光心与物体之间，则有中心透视模型：

![小孔](http://otbwgn2nv.bkt.clouddn.com/c7d090d8018cfae40527a6f5d0e5feb9.png)

可得：

![](http://otbwgn2nv.bkt.clouddn.com/2780c41b044b11200613f679c97ec9c4.png)

齐次坐标下有：

![](http://otbwgn2nv.bkt.clouddn.com/3b7c10969a79e2c92611b1343c32aa89.png)

总体上看，透视投影将相机坐标系中的点投影到理想图像坐标系，其变换过程只与相机焦距 f 有关。

### 畸变校正
理想的针孔成像模型确定的坐标变换关系均为线性的，而实际上，现实中使用的相机由于镜头中镜片因为光线的通过产生的不规则的折射，镜头畸变（lens distortion）总是存在的，即根据理想针孔成像模型计算出来的像点坐标与实际坐标存在偏差。畸变的引入使得成像模型中的几何变换关系变为非线性，增加了模型的复杂度，但更接近真实情形。畸变导致的成像失真可分为径向失真和切向失真两类：

![](http://otbwgn2nv.bkt.clouddn.com/50da8ca21000a23c748c76ef05c11352.png)

畸变类型很多，总体上可分为径向畸变和切向畸变两类，径向畸变的形成原因是镜头制造工艺不完美，使得镜头形状存在缺陷，包括枕形畸变和桶形畸变等，可以用如下表达式来描述：

![](http://otbwgn2nv.bkt.clouddn.com/8d8ed2c69ab681cbf26e1107aae6f1ef.png)

切向畸变又分为薄透镜畸变和离心畸变等，薄透镜畸变则是因为透镜存在一定的细微倾斜造成的；离心畸变的形成原因是镜头是由多个透镜组合而成的，而各个透镜的光轴不在同一条中心线上。切向畸变可以用如下数学表达式来描述：

![](http://otbwgn2nv.bkt.clouddn.com/cb54b83eaf68817ae74384c6fb03696d.png)

在引入镜头的畸变后，成像点从理想图像坐标系到真实图像坐标系的变换关系可以表示为：

![](http://otbwgn2nv.bkt.clouddn.com/95a39188156939dab30d20ad4a418cac.png)

实际计算过程中，如果考虑太多高阶的畸变参数，会导致标定求解的不稳定。

### 数字化图像
光线通过相机镜头后最终成像在感光阵列(CCD或CMOS)上，然后感光阵列将光信号转化为电信号，最后形成完整的图像。我们用dx和dy分别表示感光阵列的每个点在x和y方向上物理尺寸，即一个像素是多少毫米，这两个值一般比较接近，但由于制造工艺的精度问题，会有一定误差，同样的，感光阵列的法向和相机光轴也不是完全重合，即可以看作成像平面与光轴不垂直。
![](http://otbwgn2nv.bkt.clouddn.com/8d88f7c57ed53c087c9d2138c43f08e6.png)

我们用仿射变换来描述这个过程，如上图，O点是图像中心点，对应图像坐标(u0，v0)，Xd - Yd是真实图像坐标系，U-V是数字化图像坐标系，有：

![](http://otbwgn2nv.bkt.clouddn.com/ef4348cdde922a5537937d10821cc703.png)

齐次坐标下有：

![](http://otbwgn2nv.bkt.clouddn.com/b749d14c0e8c5d9072682ea7f9b0f868.png)

上式中的变换矩阵即为相机的内参数矩阵 K，其描述了相机坐标系中点到二维图像上点的变换过程。

综上所述，在不考虑镜头畸变的情况下，相机的整个成像过程可表示为，其中R,T为旋转矩阵和平移矩阵，为相机的外参数：

![](http://otbwgn2nv.bkt.clouddn.com/4d8fa276198b8e4437e8b4f7048cb349.png)

若考虑畸变，则为

![](http://otbwgn2nv.bkt.clouddn.com/459e6e6b8ff1fb1cd8a529b6abaa2a0a.png)

# 相机标定的步骤
1. 准备标定图片
2. 对每一张标定图片，提取角点信息
3. 对每一张标定图片，进一步提取亚像素角点信息
4. 相机标定
5. 查看标定效果——利用标定结果对棋盘图进行矫正

# 基于OpenCV的相机标定实现
## 定义全局变量
**首先定义全局变量**
```
#include "opencv2/core/core.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/calib3d/calib3d.hpp"
#include "opencv2/highgui/highgui.hpp"
#include <iostream>
#include <fstream>
#include "Calibration.h"

using namespace cv;
using namespace std;

int image_count = 0;						// 图像数量
Size board_size = Size(6, 4);				// 标定板上每行、列的角点数

Size image_size;											// 图像的尺寸
vector<vector<Point2f>> image_points_seq;					// 保存检测到的所有角点
Mat cameraMatrix = Mat(3, 3, CV_32FC1, Scalar::all(0));		// 摄像机内参数矩阵
Mat distCoeffs = Mat(1, 5, CV_32FC1, Scalar::all(0));		// 摄像机的5个畸变系数：k1,k2,p1,p2,k3

void findCheese(string filename);
void myCalibration(string filename);
void correctFile();
```
## 读入图片并寻找角点
**然后读入储存图片路径的文本文件，读入图片并且寻找图片角点**
```
void findCheese(string filename) {
	vector<Point2f> image_points_buf;  // 缓存每幅图像上检测到的角点
	cout << "开始提取角点";
	ifstream fin(filename);

	while (getline(fin, filename))
	{
		image_count++;
		cout << "Solve image = " << image_count << endl;
		Mat imageInput = imread(filename);
		if (image_count == 1)  //读入第一张图片时获取图像宽高信息,标定需要
		{
			image_size.width = imageInput.cols;
			image_size.height = imageInput.rows;
			cout << "image_size.width = " << image_size.width << endl;
			cout << "image_size.height = " << image_size.height << endl;
		}

		/* 提取角点 */
		if (0 == findChessboardCorners(imageInput, board_size, image_points_buf))
		{
			cout << "can not find chessboard corners!\n"; //找不到角点
			exit(1);
		}
		else
		{
			Mat view_gray;
			cvtColor(imageInput, view_gray, CV_RGB2GRAY);
			/* 亚像素精确化 */
			find4QuadCornerSubpix(view_gray, image_points_buf, Size(5, 5)); //对粗提取的角点进行精确化
																			//cornerSubPix(view_gray,image_points_buf,Size(5,5),Size(-1,-1),TermCriteria(CV_TERMCRIT_EPS+CV_TERMCRIT_ITER,30,0.1));
			image_points_seq.push_back(image_points_buf);  //保存亚像素角点

			//drawChessboardCorners(view_gray, board_size, image_points_buf, false); //用于在图片中标记角点
			//imshow("Camera Calibration", view_gray);//显示图片
			//waitKey(500);//暂停0.5S		
		}
	}
	cout << "角点提取完成！\n";
}
```
## 进行标定，寻找内参数与畸变参数
```
void myCalibration(string filename) {

	cout << "开始标定………………";
	/*棋盘三维信息*/
	Size square_size = Size(10, 10);  /* 实际测量得到的标定板上每个棋盘格的大小 */
	vector<vector<Point3f>> object_points; /* 保存标定板上角点的三维坐标 */
	vector<Mat> tvecsMat;  /* 每幅图像的旋转向量 */
	vector<Mat> rvecsMat; /* 每幅图像的平移向量 */
	/* 初始化标定板上角点的三维坐标 */
	int i, j, t;
	for (t = 0; t<image_count; t++)
	{
		vector<Point3f> tempPointSet;
		for (i = 0; i<board_size.height; i++)
		{
			for (j = 0; j<board_size.width; j++)
			{
				Point3f realPoint;
				/* 假设标定板放在世界坐标系中z=0的平面上 */
				realPoint.x = i*square_size.width;
				realPoint.y = j*square_size.height;
				realPoint.z = 0;
				tempPointSet.push_back(realPoint);
			}
		}
		object_points.push_back(tempPointSet);
	}

	/* 开始标定 */
	calibrateCamera(object_points, image_points_seq, image_size, cameraMatrix, distCoeffs, rvecsMat, tvecsMat, 0);
	cout << "标定完成！\n";

	//保存标定结果  
	ofstream fout(filename);  // 保存标定结果的文件
	std::cout << "开始保存定标结果………………" << endl;
	fout << "相机内参数矩阵：" << endl;
	fout << cameraMatrix << endl << endl;
	fout << "畸变系数：\n";
	fout << distCoeffs << endl << endl << endl;

	std::cout << "完成保存" << endl;
	fout << endl;
}
```

## 进行图像矫正
```
void correctFile() {
	Mat mapx = Mat(image_size, CV_32FC1);
	Mat mapy = Mat(image_size, CV_32FC1);
	Mat R = Mat::eye(3, 3, CV_32F);
	std::cout << "保存矫正图像" << endl;
	string imageFileName;
	std::stringstream StrStm;
	for (int i = 0; i != image_count; i++)
	{
		std::cout << "Save " << i + 1 << "..." << endl;
		initUndistortRectifyMap(cameraMatrix, distCoeffs, R, cameraMatrix, image_size, CV_32FC1, mapx, mapy);
		StrStm.clear();
		imageFileName.clear();
		string filePath = "C:\\Users\\whatseven\\Desktop\\project\\C++\\Calibration\\Calibration\\image\\chess";
		StrStm << i + 1;
		StrStm >> imageFileName;
		filePath += imageFileName;
		filePath += ".bmp";
		Mat imageSource = imread(filePath);
		Mat newimage = imageSource.clone();
		//另一种不需要转换矩阵的方式
		//undistort(imageSource,newimage,cameraMatrix,distCoeffs);
		remap(imageSource, newimage, mapx, mapy, INTER_LINEAR);
		StrStm.clear();
		filePath.clear();
		StrStm << i + 1;
		StrStm >> imageFileName;
		imageFileName += "_d.jpg";
		imwrite(imageFileName, newimage);
	}
	std::cout << "保存结束" << endl;
}
```

## 调用函数
```
void main()
{
	string inputFile = "calibdata.txt";
	string outputFile = "caliberation_result.txt";

	findCheese(inputFile);			//找到角点

	myCalibration(outputFile);		//标定，找到参数矩阵及畸变系数

	correctFile();					//储存矫正后图片

	return;
}
```

## 标定结果
![](http://otbwgn2nv.bkt.clouddn.com/c28b1b2342c202e556e4cc31b1a646fd.png)

**[Github项目地址](https://github.com/whatseven/Calibration.git)**

# 利用Matlab进行标定
## CameraCalibrator工具
1. 在Matlab命令行里，输入`cameraCalibrator`打开标定程序

2. 选择`Add images`将图片导入

3. 单击`Calibration`即可开始标定，再点击`Export Camera Parameters`即可导出相机参数
![](http://otbwgn2nv.bkt.clouddn.com/32b1e24f203e5bbad6795546093b9db7.png)
