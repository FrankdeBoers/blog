15年在合工大实验室，给安徽省交通厅做的项目。 通过分析交通路口摄像头抓拍的图片，分析车辆中的驾驶员是否系了安全带。

主要思路：通过客户提供的图片，前期先手动筛选出质量较好的图片，然后截取出司机系了安全带的区域照片，作为正向图片，其他区域作为负向图片。 
然后使用adaboost训练集，训练出一个可信的特征集。  

后续输入新的图片，抓取特征值，与已有的特征集比较，最终输出是否系了安全带。

~~~c++
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/nonfree/features2d.hpp>
#include <opencv2/legacy/legacy.hpp>

int main()
{
    cv::Mat ima1=cv::imread("D:\\DEMO\\penpal.jpg");
    cv::Mat ima2=cv::imread("D:\\DEMO\\res.jpg");

    // 检测surf特征点
    cv::vector<cv::KeyPoint> kp1 , kp2 ; 
    /*SurfFeatureDetector detector( 400 ) ;
    detector.detect( image1 , kp1 );
    detector.detect( image2 , kp2 );*/
    cv::SiftFeatureDetector sift( 2500 );
    sift.detect(ima1 , kp1);
    sift.detect(ima2 , kp2);
    //显示surf检测后的结果
    cv::Mat dstimg2;
    drawKeypoints( ima2 , kp2 , dstimg2 );
    imshow("image2 keypoints" , dstimg2 );
    cv::Mat dstimg1;
    drawKeypoints( ima1 , kp1 , dstimg1 );
    imshow("image1 keypoints" , dstimg1 );
    //输出surf检测的结果
    cv::vector<cv::KeyPoint> :: iterator kpCounts ;
    for( kpCounts = kp2.begin() ; kpCounts != kp2.end() ; kpCounts++ )
    {
        std::cout<<"angle:"<<kpCounts->angle<<"\t"  <<
            kpCounts->class_id<<"\t"<<kpCounts->octave<<"\t"<<
            kpCounts->pt<<"\t"<<kpCounts->response<<"\n";
    }
    // 描述surf特征点
    cv::SurfDescriptorExtractor surfDesc ;
    cv::Mat descriptros1 , descriptros2 ;
    surfDesc.compute( ima1 , kp1 , descriptros1 );
    surfDesc.compute( ima2 , kp2 , descriptros2 );
    // 计算匹配点数
    cv::BruteForceMatcher<cv::L2<float>>matcher ;
    cv::vector<cv::DMatch> matches ;
    matcher.match( descriptros1 , descriptros2 , matches );
    cv::Mat imageMatches;
    drawMatches( ima1 ,//输入图片1
        kp1 ,//输入图片1的提取的特征点
        ima2 , //输入图片2
        kp2 , //输入图片2的提取出的特征点
        matches ,//使用match函数获得的匹配特征点
        imageMatches ,//输出图片
        cv::Scalar(255,0,0)//画线颜色
        );
    imshow("image2" , imageMatches);
    cv::waitKey( 0 );
    return 0;
}
~~~
