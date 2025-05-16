---
title: CS61C FA20 Project1
description: 关于CS61C第1个项目的个人解法
date: 2025-05-16
categories:
    - CS61C
    - 解题思路
tags:
    - C
    - 学习笔记
comment: true
---

## Background

### PPM Format

题目在这里向我们介绍了一个图像存储格式 **PPM**，它由格式、宽度高度、颜色值范围以及一系列像素，其中每个像素由三个数字表示，分别代表 R G B，使用 vscode 的同学可以下载 ppm 的插件来看这些数字是如何转变为一个图像的。

### File I/O

对于文件的 IO，这里给出了三个函数

`FILE *fp = fpen("diary.txt", "r")` 这代表以只读的方式来读取文件的内容。

`fscanf()` 这可以从将文件的内容写入我们创建的变量，其中会以空格、换行符等为分界线。

`fclose()` 关闭文件。

## Part A1

本题需要完成三个函数，读取一个 ppm 文件的内容，并将其写入一个新文件，同时存在内存管理。

### `readData()`

本函数需要打开一个 ppm 文件，并将其生成一个 Image 对象。

首先要读取它的头部文件，也就是格式、高度宽度、颜色值范围，这些都很简单，不赘述。

之后，要开始读取 Image 内的内容，Image 这个结构体存在三个属性，其中两个代表的是高度与宽度，另一个代表每个像素的 Color，所以对于高度与宽度直接赋值即可，不用进行内存管理，但其中还包含不确定的 Color，因此在这里我们要动态分配内存。

对于每个 Image，其宽度与高度是不确定的，所以不能简单的创建一个定量的二维数组，应该要使用动态分配内存的二维数组。在 C 语言当中，动态分配二维数组有一个很常见的方法，先指定一个每一行的指针，再由这个指针指向每一列，这就能很方便的访问二维数组里面的内容了。

对于每一个 Color，其中又包含了三个值，R G B，要对他们进行赋值，很简单，不赘述。

```c
Image *readData(char *filename) 
{
	FILE *fp = fopen(filename, "r");
	char format[3];
	fscanf(fp, "%s", format);
	int width, height;
	fscanf(fp, "%d %d", &width, &height);
	int scale;
	fscanf(fp, "%d", &scale);

	Image *img = (Image*) malloc(sizeof(Image));
	img->rows = height;
	img->cols = width;
	
	img->image = (Color**) malloc(sizeof(Color*) * height);
	for (int i = 0; i < height; i++) {
		img->image[i] = (Color*) malloc(sizeof(Color) * width);
	}

	for (int i = 0; i < height; i++) {
		for (int j = 0; j < width; j++) {
			int r, g, b;
			fscanf(fp, "%d %d %d", &r, &g, &b);
			img->image[i][j].R = r;
			img->image[i][j].G = g;
			img->image[i][j].B = b;
		}
	}
	fclose(fp);
	return img;
}
```

### `writeData()`

给定一个 Image 对象，要将其打印出来，整体来说不难，有几个点需要注意

1. 遇到 Image 对象没有的属性，直接以字符串的方式打印即可
2. 在打印 RGB 值时，要保证三个字符的格式化 `%3d`
3. 在打印完一个 Color 的内容后要记得空三个空格，但每一行最后一个 Color 的内容后不换行，要注意区分
```c
void writeData(Image *image)
{
	//YOUR CODE HERE
	printf("P3\n");
	printf("%d %d\n", image->cols, image->rows);
	printf("255\n");

	for (int i = 0; i < image->rows; i++) {
		for (int j = 0; j < image->cols; j++) {
			printf("%3d %3d %3d", 
				image->image[i][j].R, 
				image->image[i][j].G, 
				image->image[i][j].B);
				if (j < image->cols - 1) {
					printf("   ");
				}
		}
		printf("\n");
	}
}
```

### `freeImage()`

将 Image 分配的内存释放出来，要注意的是先释放最后分配的内存，这样一步一步来释放，像我们之前有一个指向每一列的指针，这个指针指向每一列的内容，那我们就要把这个指针最先释放

```c
void freeImage(Image *image)
{
	for (int i = 0; i < image->rows; i++) {
		free(image->image[i]);
	}
	free(image->image);
	free(image);
}
```
此时如果正确实现了代码，并且下载了 vscode ppm 的扩展，你应该可以看见一个老人的图片。
## Part A2

题目向我们介绍了一个新的知识**Steganography**，这指的是我们可以将信息隐藏在图像的底部位，具体步骤为修改每个像素中 RGB 的 B 值的最低位，如果这个最低位是 0，那我们就将其设置为黑色，如果是 1，那就设置为白色。

所以我们可以看出这里还需要运用**按位**的操作，来得到最低位的值。

### `evaluateOnePixel()`

将 Image 里特定位置的像素设置为隐藏位，步骤大致与上文提到的无异，需要注意的是

- 这里是创建一个新的像素，而不是在原像素上做文章，所以需要重新分配内存
```c
Color *evaluateOnePixel(Image *image, int row, int col)
{
	Color *color = (Color*) malloc(sizeof(Color));
	uint8_t new_B = image->image[row][col].B;
	new_B &= 1;
	if (new_B) {
		color->R = 255;
		color->G = 255;
		color->B = 255;
	} else {
		color->R = 0;
		color->G = 0;
		color->B = 0;
	}
	return color;
}
```

### `steganography()`

给定一个特定的图像，将其按上述提过的创建一个新图像。

很明显，我们需要用到上文完成的函数，需要注意的是，这里依然是创建一个新图像，而不是在原图像做文章，所以需要内存管理，具体步骤类似于二维数组的创建，然后将二维数组特殊位用函数重新生成就好。

```c
Image *steganography(Image *image)
{
	Image *img = (Image*) malloc(sizeof(Image));
	uint32_t row = image->rows, col = image->cols;
	
	img->rows = row;
	img->cols = col;
	
	img->image = (Color**) malloc(sizeof(Color*) * row);
	for (int i = 0; i < row; i++) {
		img->image[i] = (Color*) malloc(sizeof(Color) * col);
	}

	for (int i = 0; i < row; i++) {
		for (int j = 0; j < col; j++) {
			Color *new_color = evaluateOnePixel(image, i, j);
			img->image[i][j] = *new_color;
			free(new_color);
		}
	}
	return img;
}
```

### `main`

本题要求我们用**命令行**的内容完成上述的操作，给定一个文件，将其按隐藏码的关系生成一个新文件，需要注意的是命令行的使用，比如 `argc` 代表什么，怎么用，以及正确释放内存。

```c
int main(int argc, char **argv)
{
	if (argc != 2) {
		return -1;
	}
	
	char *filename = argv[1];
	Image *img = readData(filename);
	Image *new_img = steganography(img);
	writeData(new_img);
	freeImage(new_img);
	freeImage(img);
	
	return 0;
}
```

代码全部完成后，就会发现新生成的图片有一个 *secret message* 的标志。

## Part B

```c
Color *evaluateOneCell(Image *image, int row, int col, uint32_t rule)
{
    int rows = image->rows, cols = image->cols;
    Color **currPix = image->image + row * cols + col;
    Color *newPix = (Color *) malloc(sizeof(Color));
    if (!newPix) exit(-1);

    int dx[8] = {-1, -1, -1, 0, 0, 1, 1, 1};
    int dy[8] = {-1, 0, 1, -1, 1, -1, 0, 1};
    int aliveNeighborsR = 0, aliveNeighborsG = 0, aliveNeighborsB = 0;
    for (int i = 0; i < 8; i++) {
        int neighborRow = (row + dx[i] + rows) % rows;
        int neighborCol = (col + dy[i] + cols) % cols;
        Color **neighbor = image->image + neighborRow * cols + neighborCol;
        if ((*neighbor)->R == 255) aliveNeighborsR++;
        if ((*neighbor)->G == 255) aliveNeighborsG++;
        if ((*neighbor)->B == 255) aliveNeighborsB++;
    }

    int maskR = ((*currPix)->R == 255) * 9 + aliveNeighborsR;
    int maskG = ((*currPix)->G == 255) * 9 + aliveNeighborsG;
    int maskB = ((*currPix)->B == 255) * 9 + aliveNeighborsB;
    newPix->R = (rule & (1 << maskR)) ? 255 : 0;
    newPix->G = (rule & (1 << maskG)) ? 255 : 0;
    newPix->B = (rule & (1 << maskB)) ? 255 : 0;

    return newPix;
}

Image *life(Image *image, uint32_t rule)
{
	Image *img = (Image*) malloc(sizeof(Image));
	uint32_t row = image->rows, col = image->cols;

	img->rows = row;
	img->cols = col;

	img->image = (Color**) malloc(sizeof(Color*) * row);
	for (int i = 0; i < row; i++) {
		img->image[i] = (Color*) malloc(sizeof(Color) * col);
	}

	for (int i = 0; i < row; i++) {
		for (int j = 0; j < col; j++) {
			Color *new_color = evaluateOneCell(image, i, j, rule);
			img->image[i][j] = *new_color;
			free(new_color);
		}
	}
	return img;
}

int main(int argc, char **argv)
{
	if (argc != 3) {
		printf("usage: ./gameOfLife filename rule\n");
		printf("filename is an ASCII PPM file (type P3) with maximum value 255.\n");
		printf("rule is a hex number beginning with 0x; Life is 0x1808.\n");
	}

	char *filename = argv[1];
	uint32_t rule = argv[2];
	Image *img = readData(filename);
	Image *new_img = life(img, rule);
	writeData(new_img);
	freeImage(new_img);
	freeImage(img);
	
	return 0;
}
```