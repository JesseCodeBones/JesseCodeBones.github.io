# libjpeg打开和写入jpeg文件
### 流程
1.1分配和初始化一个JPEG decompression结构体

    struct jpeg_decompress_struct cinfo;
     jpeg_create_decompress(&cinfo);

1.2指定待解压的源文件

     infile = fopen(argv[1], "rb");
     jpeg_stdio_src(&cinfo, infile);
1.3使用jpeg_read_header获得jpg信息

     jpeg_read_header(&cinfo, TRUE);
     cinfo.image_width//jpg图片的宽度
     cinfo.image_height//jpg图片的高度
     cinfo.num_components//每个图片的像素值由几部分组成，如rgb就是3部分
1.4设置解压参数

    cinfo.scale_num//scale_num/scale_denom为图像相对于原图的缩放比例
    cinfo.scale_denom
1.5启动解压

   jpeg_start_decompress(&cinfo);//启动jpg解压  

1.6 循环调用jpeg_read_scanlines  

  row_stride = cinfo.output_width * cinfo.output_components;//解压后图像的一行的字节长度
  buffer = malloc(row_stride);
  jpeg_read_scanlines(&cinfo, &buffer, 1);//读取图片的一行到buffer中去

1.7释放获取的资源  
 jpeg_finish_decompress(&cinfo);
 jpeg_destroy_decompress(&cinfo);

### 实例代码
```C++
#include <iostream>
#include "jpeglib.h"
#include <setjmp.h>
#include <vector>


std::vector<unsigned char> compressed_target;
struct my_error_mgr {
  struct jpeg_error_mgr pub;    /* "public" fields */

  jmp_buf setjmp_buffer;        /* for return to caller */
};

typedef struct my_error_mgr *my_error_ptr;
METHODDEF(void)
my_error_exit(j_common_ptr cinfo)
{
  my_error_ptr myerr = (my_error_ptr)cinfo->err;
  (*cinfo->err->output_message) (cinfo);
  longjmp(myerr->setjmp_buffer, 1);
}

METHODDEF(int)
do_read_JPEG_file(struct jpeg_decompress_struct *cinfo, const char *filename)
{
  
  struct my_error_mgr jerr;
  FILE *infile;                 /* source file */
  JSAMPARRAY buffer;            /* Output row buffer */
  int row_stride;               /* physical row width in output buffer */

  if ((infile = fopen(filename, "rb")) == NULL) {
    fprintf(stderr, "can't open %s\n", filename);
    return 0;
  }

  cinfo->err = jpeg_std_error(&jerr.pub);
  jerr.pub.error_exit = my_error_exit;
  if (setjmp(jerr.setjmp_buffer)) {
    jpeg_destroy_decompress(cinfo);
    fclose(infile);
    return 0;
  }
  jpeg_create_decompress(cinfo);
  jpeg_stdio_src(cinfo, infile);
  (void)jpeg_read_header(cinfo, TRUE);
  /*这里是设置缩放比列，这里设置1/4 表示原图1920*1080会缩放到480*270,若不设置则按原图大小尺寸*/
  // cinfo.scale_num  = 1;
  // cinfo.scale_denom = 4;
  (void)jpeg_start_decompress(cinfo);
  row_stride = cinfo->output_width * cinfo->output_components;
  buffer = (*cinfo->mem->alloc_sarray)
                ((j_common_ptr)cinfo, JPOOL_IMAGE, row_stride, 1);

  while (cinfo->output_scanline < cinfo->output_height) {
    (void)jpeg_read_scanlines(cinfo, buffer, 1);
    std::copy(buffer[0], buffer[0] + (cinfo->output_width*3), std::back_inserter(compressed_target));
  }

  (void)jpeg_finish_decompress(cinfo);
  jpeg_destroy_decompress(cinfo);
  fclose(infile);
  return 1;
}

GLOBAL(void)
write_JPEG_file(char const *const filename, int quality, int orgWidth, int orgHeight)
{
 
  struct jpeg_compress_struct cinfo;
  
  struct jpeg_error_mgr jerr;
  /* More stuff */
  FILE *outfile;                /* target file */
  JSAMPROW row_pointer[1];      /* pointer to JSAMPLE row[s] */
  int row_stride;               /* physical row width in image buffer */

  cinfo.err = jpeg_std_error(&jerr);
  jpeg_create_compress(&cinfo);

  if ((outfile = fopen(filename, "wb")) == NULL) {
    fprintf(stderr, "can't open %s\n", filename);
    exit(1);
  }
  jpeg_stdio_dest(&cinfo, outfile);

  cinfo.image_width = orgWidth;      /* image width and height, in pixels */
  cinfo.image_height = orgHeight;
  cinfo.input_components = 3;           /* # of color components per pixel */
  cinfo.in_color_space = JCS_RGB;       /* colorspace of input image */
  jpeg_set_defaults(&cinfo);
  jpeg_set_quality(&cinfo, quality, TRUE /* limit to baseline-JPEG values */);

  jpeg_start_compress(&cinfo, TRUE);
  row_stride = orgWidth * 3; /* JSAMPLEs per row in image_buffer */

  while (cinfo.next_scanline < cinfo.image_height) {
    row_pointer[0] = &compressed_target[cinfo.next_scanline * row_stride];
    (void)jpeg_write_scanlines(&cinfo, row_pointer, 1);
  }

  jpeg_finish_compress(&cinfo);
  fclose(outfile);
  jpeg_destroy_compress(&cinfo);
}

int main(int, char**) {
    jpeg_decompress_struct jds;
    const char *fileName = "/home/jesse/Downloads/test.jpg";
    do_read_JPEG_file(&jds, fileName);
    const char*const targetName = "./testjesse.jpeg";
    write_JPEG_file(targetName, 100, jds.image_width, jds.image_height);
    
}

```

CMakeLists  
```
cmake_minimum_required(VERSION 3.0.0)
project(libjpegdemo VERSION 0.1.0)

include(CTest)
enable_testing()

add_executable(libjpegdemo main.cpp)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
include(FetchContent)
FetchContent_Declare(
  libjpeg
  GIT_REPOSITORY https://github.com/libjpeg-turbo/libjpeg-turbo.git
  GIT_TAG 4f7a8afbb7839a82dd357b5128cfb04dac6a5986
)
FetchContent_MakeAvailable(libjpeg)
target_link_libraries(libjpegdemo PUBLIC jpeg-static)
target_include_directories(libjpegdemo PUBLIC ${libjpeg_BINARY_DIR} ${libjpeg_SOURCE_DIR})
```