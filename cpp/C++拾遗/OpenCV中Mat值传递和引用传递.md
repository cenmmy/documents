# OpenCV中Mat值传递和引用传递

![mat结构体](/home/cenmmy/Documents/documents/cpp/C++拾遗/mat结构体.png)

当我们使用值传递的时候，只能修改结构体中的指针成员。比如mat中存储的数据。

当我们使用引用传递的时候，我们可以修改所有的数据。



比如，有个函数可以对图像进行颜色变换

```C++
void CvtColorImgOperator::operate(cv::Mat img) {
    cv::cvtColor(img, img, cv::COLOR_BGR2GARY);
}
```

这个函数中我们对图像进行灰度化处理，经过灰度化处理之后的图像，通道数从3变化为了1。在函数中打印我们可以得道img的通道数为1。

但是如果在函数之外打印，则图像的通道数还是为3。

相反如果使用引用传递则没有这种情况，灰度化之后的图像的通道数在函数内还是在函数外都是1。