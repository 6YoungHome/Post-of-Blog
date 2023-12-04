---
title: 【编程学习】使用Python处理pdf文件
typora-root-url: 【编程学习】使用Python处理pdf文件
cover: /img/blog_img/4.png
description: 使用Python处理pdf文件：pdf解密、合并、拆分以及图片转pdf
tags:
  - Python
  - 有用的代码
categories:
  - 编程学习
  - Python
abbrlink: 2e2de884
date: 2023-07-01 01:44:38
---



虽然对于pdf的基本处理通过WPS就能处理了，但是作为一个学Python的码农，用Python处理一下也是合理的orz  

(○｀ 3′○)

## 基本库的配置

实现pdf处理功能主要用到以下的一些Python库：

```
pip install PyPDF2
pip install fitz
pip install pikepdf
```



## pdf解密

总有一些pdf会设有密码，影响我们观看，其实用Python可以很方便的去除pdf的密码。

（所以别给奇怪的文件设密码了，没用(￣y▽,￣)╭ ）

方法很简单，只需要用到pikepdf库，就可以完成。

```python
import pikepdf

def unlock(inputfile, outputfile):
    pdf = pikepdf.open(inputfile)
    pdf.save(outputfile)
```



## pdf拼接

```python
def linkpdfs(pdfdir, newfile='newfile.pdf'):
    '''拼接目标目录中的所有pdf文件'''
    files = os.listdir(pdfdir)#列出目录中的所有文件
    merger = PdfFileMerger()
    for file in files: #从所有文件中选出pdf文件合并
        if file[-4:] == ".pdf":
            merger.append(open(pdfdir+'/'+file, 'rb'))
    with open(newfile, 'wb') as fout:  #输出文件为newfile.pdf
        merger.write(fout)
```



## 图片转pdf

```python
def png2pdf(path='.'):
    '''把图片变为PDF'''
    for name in glob.glob(os.path.join(path, '*.png')):
        imgdoc = fitz.open(name)
        pdfbytes = imgdoc.convertToPDF()    # 使用图片创建单页的 PDF
        imgpdf = fitz.open("pdf", pdfbytes)
        imgpdf.save(name[:-4] + '.pdf')
```



## 节选pdf

```python
def cutpdf(inputfile, start_page, end_page, outputfile='output.pdf'):
    '''节选pdf文件'''
    output = PdfFileWriter()
    pdf_file = PdfFileReader(open(inputfile,"rb"))
    for i in range(start_page,end_page):
        output.addPage(pdf_file.getPage(i))
    outputStream = open(outputfile,"wb")
    output.write(outputStream)
```















