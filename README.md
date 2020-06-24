# cpp gstreamer pipelines

Create c++ OpenCV compatible gstreamer pipelines with fine tuned control.

## Features

-   No OpenCV dependencies
-   Single Header-only library
-   Fine tuned control of how opencv will receive the stream

## C++ Example

```cpp
#include <iostream>

#include "gstreamer_pipelines.hpp"
#include <opencv2/opencv.hpp>

int main()
{
    // Make the pipeline object with the streamer
    OpencvGStreamerPipeline opencvGSpipeline("nvarguscamerasrc");
    // Add a first source to process the streamer
    opencvGSpipeline.addSource(
        "video/x-raw(memory:NVMM), width=(int)1280, height=(int)720, "
        "format=(string)I420, framerate=(fraction)30/1");

    // Add a sink and a source to process the sink
    opencvGSpipeline.addSink("nvvidconv");
    opencvGSpipeline.addSource("video/x-raw, format=(string)BGRx");

    // Add a sink and source together
    std::string secondSink = "videoconvert";
    std::string secondSource = "video/x-raw, format=(string)BGR";
    opencvGSpipeline.addElement(secondSink, secondSource);

    // Add the final appSink for opencv
    opencvGSpipeline.addSink("appsink");

    // Get the pipeline string to pass to opencv
    std::string GSpipeline = opencvGSpipeline.getPipelineString();
    std::cout << "G-Streamer pipeline is: " << GSpipeline << std::endl;

    // Opencv capture
    cv::VideoCapture cap(GSpipeline, cv::CAP_GSTREAMER);
    if (!cap.isOpened())
    {
        std::cout << "Failed to open camera." << std::endl;
        return (-1);
    }

    cv::namedWindow("CSI Camera", cv::WINDOW_AUTOSIZE);
    cv::Mat frame;

    std::cout << "Hit ESC to exit" << std::endl;
    while (true)
    {
        cap >> frame;
        if (frame.empty())
        {
            std::cerr << "Frame not grabbed" << std::endl;
            exit(-2);
        }

        cv::imshow("CSI Camera", frame);
        int keycode = cv::waitKey(30) & 0xff;
        if (keycode == 27)
            break;
    }

    // Cleanup
    cap.release();
    cv::destroyAllWindows();
    return 0;
}

```

### Compile Example File

```sh
g++ -g -Wall example.cpp -o example -std=c++11 `pkg-config --cflags --libs opencv4`
```
