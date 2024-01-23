实际上，stb_image和UE5没什么关系，是GitHub一个开源图像处理工具，放在此处仅为了方便。

```c
#include <iostream>
#include <vector>
#define STB_IMAGE_IMPLEMENTATION // 这个宏不能少很关键
#include "stb_image.h" // 仅需包含头文件，没有静/动态库，源码在头文件里

using namespace std;

int main() {
    // 文件路径
    const char* filePath = "R.png";

    // 使用stb_image加载PNG图像
    int width, height, channels;
    stbi_uc* image = stbi_load(filePath, &width, &height, &channels, STBI_rgb_alpha);

    if (!image) {
        std::cout << "Failed to load image: " << filePath << std::endl;
        return EXIT_FAILURE;
    }

    cout << "load success" << endl;
    cout << width << " " << height << " " << channels << endl;

    // 将图像数据存储到std::vector中
    std::vector<stbi_uc> imageVector(image, image + (width * height * STBI_rgb_alpha));

    // 在此处，你可以使用imageVector进行后续的图像处理或操作

    // 释放stb_image分配的内存
    stbi_image_free(image);

    return EXIT_SUCCESS;
}

```

