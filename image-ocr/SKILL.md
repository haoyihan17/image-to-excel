---
name: "image-ocr"
description: "从图片中识别并提取文字和表格数据，支持中文OCR。当用户需要从图片（如统计年鉴截图、扫描件）中提取数据、识别表格、或批量处理图片中的文字时调用此 Skill。"
---

# 图片数据识别 (Image OCR)

从图片中识别文字和表格数据，特别针对中文统计年鉴等数据表格图片优化。

## 使用场景

- 从统计年鉴的 JPG/PNG 表格图片中提取数据
- 批量识别图片中的中文文字和数字
- 将图片中的表格转换为结构化数据（CSV/Excel）

## 工作流程

### 第一步：安装依赖

```powershell
pip install pytesseract Pillow openpyxl pandas requests
```

还需要安装 Tesseract OCR 引擎：
- Windows: 下载安装 https://github.com/UB-Mannheim/tesseract/wiki
- 安装时勾选中文语言包 `Chinese (Simplified)`
- 默认安装路径: `C:\Program Files\Tesseract-OCR\tesseract.exe`

### 第二步：下载目标图片

从统计年鉴网站下载图片：

```python
import requests
import os

def download_image(url, save_path):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    }
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        os.makedirs(os.path.dirname(save_path), exist_ok=True)
        with open(save_path, 'wb') as f:
            f.write(response.content)
        print(f"Downloaded: {save_path}")
        return True
    else:
        print(f"Failed: {url} (status: {response.status_code})")
        return False
```

### 第三步：OCR 识别图片中的文字

```python
import pytesseract
from PIL import Image

# 设置 Tesseract 路径（Windows）
pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'

def ocr_image(image_path, lang='chi_sim+eng'):
    """
    识别图片中的文字
    lang: 'chi_sim' 简体中文, 'chi_sim+eng' 中英文混合
    """
    image = Image.open(image_path)
    # 预处理：提高对比度
    from PIL import ImageEnhance
    enhancer = ImageEnhance.Contrast(image)
    image = enhancer.enhance(2.0)
    
    text = pytesseract.image_to_string(image, lang=lang, config='--psm 6')
    return text
```

### 第四步：解析表格数据

```python
def parse_table_from_text(text):
    """
    从 OCR 结果中解析表格数据
    统计年鉴表格通常格式为：
    年份    中央    地方    全国
    2000    xxx     xxx     xxx
    """
    lines = text.strip().split('\n')
    data = []
    for line in lines:
        # 按空格或制表符分割
        parts = line.split()
        # 过滤掉非数据行
        if parts and any(c.isdigit() for c in parts[0]):
            data.append(parts)
    return data
```

### 第五步：导出为 Excel

```python
import pandas as pd

def save_to_excel(data, headers, output_path):
    df = pd.DataFrame(data, columns=headers)
    df.to_excel(output_path, index=False)
    print(f"Saved to: {output_path}")
```

## 完整示例脚本

将以下脚本保存为 `ocr_extract.py`：

```python
import pytesseract
from PIL import Image, ImageEnhance
import requests
import os
import pandas as pd
import re

pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'

def download_image(url, save_path):
    headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'}
    resp = requests.get(url, headers=headers, timeout=30)
    if resp.status_code == 200:
        os.makedirs(os.path.dirname(save_path), exist_ok=True)
        with open(save_path, 'wb') as f:
            f.write(resp.content)
        return True
    return False

def ocr_image(image_path, lang='chi_sim+eng'):
    image = Image.open(image_path)
    enhancer = ImageEnhance.Contrast(image)
    image = enhancer.enhance(2.0)
    enhancer = ImageEnhance.Sharpness(image)
    image = enhancer.enhance(2.0)
    text = pytesseract.image_to_string(image, lang=lang, config='--psm 6 -c tessedit_char_whitelist=0123456789.')
    return text

def extract_numbers(text):
    """从OCR文本中提取所有数字"""
    numbers = re.findall(r'\d+\.?\d*', text)
    return [float(n) for n in numbers]

# 使用示例
if __name__ == '__main__':
    # 下载统计年鉴图片
    url = 'https://www.stats.gov.cn/sj/ndsj/2024/html/C07-03.jpg'
    path = 'images/C07-03.jpg'
    
    if download_image(url, path):
        text = ocr_image(path)
        print("识别结果:")
        print(text)
        
        numbers = extract_numbers(text)
        print("\n提取的数字:")
        print(numbers)
```

## 统计年鉴表格结构参考

国家统计局统计年鉴中"地方财政一般公共预算支出"相关表格：

| 年份 | 表格编号 | URL 模式 |
|------|---------|----------|
| 2024 | 7-3 中央和地方一般公共预算主要支出项目 | `https://www.stats.gov.cn/sj/ndsj/2024/html/C07-03.jpg` |
| 2024 | 7-6 分地区一般公共预算支出 | `https://www.stats.gov.cn/sj/ndsj/2024/html/C07-06.jpg` |
| 2023 | 7-3 | `https://www.stats.gov.cn/sj/ndsj/2023/html/C07-03.jpg` |
| ... | ... | ... |

URL 模式：`https://www.stats.gov.cn/sj/ndsj/{年份}/html/C{章节}-{序号}.jpg`

## 注意事项

1. Tesseract 对中文表格的识别准确率有限，建议先尝试识别纯数字列
2. 图片质量影响识别效果，可先对图片进行预处理（提高对比度、二值化）
3. 对于复杂表格，可能需要手动校验识别结果
4. 如果 Tesseract 效果不佳，可考虑使用百度OCR API 或腾讯OCR API