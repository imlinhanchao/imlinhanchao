---
author: hancel.lin
date: 2021-02-25
title: 半透明色混色算法
tags: 
    - 颜色
    - color
    - 混色
    - 算法
category: tech
layout: post
guid: urn:uuid:2157fd61-177c-4ad7-8b3c-f48fa9f1dca1
---

最近遇到一个需求，PM 希望可以在设备上呈现一个自定义的图像，上面叠加一个半透明的 Logo 。

由于底部图像是自定义的，且 Logo 位置也非固定，这就需要自行生成一个叠加图像，给到设备去显示。这时一个问题便出现了。我们需要对 Logo 的半透明区域和背景图像的颜色做混色处理。

![示例](/media/files/color-mixing-algorithm/demo.png)

<!--more-->

因此，我们要能够对每一个像素点的颜色进行混色。混色的原理其实十分简单，已知底图像素颜色为 `c1`，半透明图像素点颜色为 `c2`，透明度 alpha 值为 `a`，不透明 alpha 值为 `a'`。那么也就是混色区域的颜色值 `c`，其 `c2` 颜色保留 `a / a'`，`c1` 颜色保留 `1 - a / a'`。那么就可以得出公式：

```
c = (c1 * (a' - a) + c2 * a) / a'
```

按照这种方式分别计算 RGB 三个通道的颜色值即可。

这里附上 VC 版本的[实现](https://github.com/imlinhanchao/mix)。核心方法实现[如下](https://github.com/imlinhanchao/Mix/blob/811f3eabb197380f2c06d1e8886d4a164c5e92e8/Mix/MixDlg.cpp#L192)：

```cpp
CString CMixDlg::MixPicture( CString sPic1, CString sPic2, CPoint ptPos, CString sSave )
{
	ATL::CImage img1, img2;

	img1.Load(sPic1); img2.Load(sPic2);
	int nWidth1 = img1.GetWidth(), nHeight1 = img1.GetHeight();
	int nWidth2 = img2.GetWidth(), nHeight2 = img2.GetHeight();

	for (int i = 0; i < nWidth2; i++)
	{
		int nX = ptPos.x + i;
		if (nX > nWidth1) break;
		for (int j = 0; j < nHeight2; j++)
		{
			int nY = ptPos.y + j;
			if (nY > nHeight1) break;

            // get color and alpha from image
			COLORREF clr1 = img1.GetPixel(nX, nY);
			COLORREF clr2 = img2.GetPixel(i, j);
			BYTE* pAlpha = (BYTE*)img2.GetPixelAddress(i, j) + 3;
			int nAlpha = (*pAlpha), nRAlpha = 255 - (*pAlpha);
			
            // mixing RGB
			DWORD dwR = (GetRValue(clr2) * nAlpha + GetRValue(clr1) * nRAlpha) / 255.0 + 0.5;
			DWORD dwG = (GetGValue(clr2) * nAlpha + GetGValue(clr1) * nRAlpha) / 255.0 + 0.5;
			DWORD dwB = (GetBValue(clr2) * nAlpha + GetBValue(clr1) * nRAlpha) / 255.0 + 0.5;
			COLORREF clrFinal = RGB(dwR, dwG, dwB);

			img1.SetPixel(nX, nY, clrFinal);
		}
	}

	img1.Save(sSave);

	return sSave;
}
```
