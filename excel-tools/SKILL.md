---
name: "excel-tools"
description: "使用 Python 标准库(csv)读写 Excel 兼容的 CSV 文件，支持数据汇总、格式转换、统计分析。当用户需要创建 Excel 表格、处理 CSV 数据、或导出数据为 Excel 格式时调用此 Skill。"
---

# Excel 工具 (Excel Tools)

使用 Python 标准库（无需额外安装依赖）读写 Excel 兼容的 CSV 文件（UTF-8 BOM 编码，Excel 可直接打开）。

## 使用场景

- 将数据导出为 Excel 可打开的 CSV 文件
- 读取 CSV/Excel 数据进行统计分析
- 数据汇总、排序、筛选
- 多表合并

## 核心代码模板

### 1. 写入 Excel 兼容的 CSV

```python
import csv
import os

def save_to_excel_csv(data_rows, headers, output_path):
    """
    保存数据为 Excel 兼容的 CSV 文件
    data_rows: [(col1, col2, ...), ...]
    headers: ['列名1', '列名2', ...]
    """
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    with open(output_path, 'w', newline='', encoding='utf-8-sig') as f:
        writer = csv.writer(f)
        writer.writerow(headers)
        writer.writerows(data_rows)
    print(f"已保存: {output_path}")
```

### 2. 读取 CSV 文件

```python
import csv

def read_csv(file_path):
    """读取 CSV 文件，返回 (headers, rows)"""
    with open(file_path, 'r', encoding='utf-8-sig') as f:
        reader = csv.reader(f)
        headers = next(reader)
        rows = [row for row in reader]
    return headers, rows
```

### 3. 数据排序

```python
def sort_data(rows, col_index, reverse=False):
    """按指定列排序"""
    return sorted(rows, key=lambda x: float(x[col_index]) if x[col_index].replace('.','').replace('-','').isdigit() else x[col_index], reverse=reverse)
```

### 4. 数据筛选

```python
def filter_data(rows, col_index, condition):
    """按条件筛选行"""
    return [row for row in rows if condition(row[col_index])]
```

### 5. 计算统计信息

```python
def calc_stats(rows, col_index):
    """计算指定列的统计信息"""
    values = [float(row[col_index]) for row in rows if row[col_index]]
    return {
        'count': len(values),
        'sum': sum(values),
        'avg': sum(values) / len(values) if values else 0,
        'max': max(values) if values else 0,
        'min': min(values) if values else 0,
    }
```

## 完整示例：创建数据报表

```python
import csv
import os

output_dir = 'output'
os.makedirs(output_dir, exist_ok=True)

# 示例数据
headers = ['年份', '数值(亿元)', '同比增长(%)']
data = [
    (2020, 210492.46, 3.3),
    (2021, 211271.54, 0.3),
    (2022, 225039.25, 6.4),
    (2023, 236354.42, 5.1),
    (2024, 243892.00, 3.2),
]

# 保存
path = os.path.join(output_dir, '报表.csv')
with open(path, 'w', newline='', encoding='utf-8-sig') as f:
    writer = csv.writer(f)
    writer.writerow(headers)
    writer.writerows(data)

print(f"报表已生成: {path}")
```

## 注意事项

1. 使用 `utf-8-sig` 编码（带 BOM），确保 Excel 能正确识别中文
2. CSV 文件可以直接用 Excel 打开，另存为 `.xlsx` 即可
3. 如果环境支持 `openpyxl`，可以直接生成 `.xlsx` 文件
4. 数字不要加千分位逗号，否则 Excel 可能识别为文本