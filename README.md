<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>基因—个性报告自动化脚本</title>
  <style>
    body { font-family: Arial, sans-serif; line-height: 1.6; padding: 20px; background: #f9f9f9; }
    h1 { color: #333; }
    pre { background: #2d2d2d; color: #f8f8f2; padding: 15px; border-radius: 4px; overflow-x: auto; }
    code { font-family: Consolas, monospace; font-size: 14px; }
    .section { margin-bottom: 30px; }
  </style>
</head>
<body>

  <h1>基因—个性报告自动化脚本</h1>

  <div class="section">
    <h2>使用说明</h2>
    <ol>
      <li>将本文件另存为 <code>report_generator.html</code>（可直接在 GitHub 仓库中查看）。</li>
      <li>在同级目录新建 <code>requirements.txt</code>，添加以下内容：<br>
        <code>pandas<br>numpy<br>matplotlib</code>
      </li>
      <li>在本地或服务器上运行：<br>
        <code>pip install -r requirements.txt<br>python report_generator.py</code>
      </li>
    </ol>
  </div>

  <div class="section">
    <h2>完整脚本：report_generator.py</h2>
    <pre><code>#!/usr/bin/env python3
"""
Report Generator: 基因—个性自动化脚本

使用方法：
1. 在同级目录创建 requirements.txt，内容：
   pandas
   numpy
   matplotlib
2. 安装依赖并运行：
   pip install -r requirements.txt
   python report_generator.py
"""

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import os

# ===== 1. 用户信息输入 =====
def get_user_info():
    gender = input("请输入您的性别（male/female/other）：").strip().lower()
    if gender not in ['male', 'female', 'other']:
        gender = 'other'
    try:
        age = int(input("请输入您的年龄：").strip())
    except:
        age = 30
    return gender, age

if __name__ == '__main__':
    gender, age = get_user_info()

    # ===== 2. 六大维度及示例原始得分 =====
    dimensions = [
        'Extraversion',
        'Agreeableness',
        'Conscientiousness',
        'Neuroticism',
        'Openness',
        'Stability'
    ]
    # 示例：实际由 PGS + G×G + 概率模型生成
    initial_scores = np.array([75, 60, 82, 45, 90, 70])

    # ===== 3. 性别与年龄调整因子 =====
    gender_factors = {
        'male':   [1.05, 0.95, 1.00, 1.10, 0.98, 1.00],
        'female': [0.95, 1.05, 1.02, 0.90, 1.02, 1.00]
    }
    def age_factor(age):
        # 将年龄映射到 [0.8, 1.2] 区间
        return max(0.8, min(1.2, 1 + (30 - age) * 0.005))

    def adjust_scores(scores, gender, age):
        if gender == 'other':
            m = np.array(gender_factors['male'])
            f = np.array(gender_factors['female'])
            factor = (m + f) / 2
        else:
            factor = np.array(gender_factors.get(gender, gender_factors['female']))
        return scores * factor * age_factor(age)

    # ===== 4. 得分调整与总分计算 =====
    adjusted_scores = adjust_scores(initial_scores, gender, age)
    total_score = round(np.mean(adjusted_scores), 1)

    # ===== 5. 绘制雷达图 =====
    angles = np.linspace(0, 2 * np.pi, len(dimensions), endpoint=False).tolist()
    values = np.concatenate((adjusted_scores, [adjusted_scores[0]]))
    angles += angles[:1]

    fig, ax = plt.subplots(figsize=(6, 6), subplot_kw=dict(polar=True))
    ax.plot(angles, values, linewidth=2)
    ax.fill(angles, values, alpha=0.25)
    ax.set_thetagrids(np.degrees(angles[:-1]), dimensions)
    ax.set_ylim(0, 100)
    ax.set_title('初步基因个性评分雷达图')
    plt.tight_layout()
    plt.show()

    # ===== 6. 输出初步报告 =====
    prelim_df = pd.DataFrame({
        'Dimension': dimensions,
        'Adjusted Score': np.round(adjusted_scores, 1)
    })
    prelim_df.loc[len(prelim_df)] = ['Total Score', total_score]
    print("\n=== 初步报告：基因个性评分表 ===")
    print(prelim_df.to_string(index=False))

    # ===== 7. 深度报告（付费版）生成 =====
    deep_report_path = "deep_report.pdf"
    with open(deep_report_path, "wb") as f:
        # TODO: 用实际深度报告内容替换占位符
        f.write(b"%PDF-1.4\n% 深度报告内容占位符\n")
    print(f"\n深度报告已生成（付费下载）：{os.path.abspath(deep_report_path)}")
    </code></pre>
  </div>

  <div class="section">
    <h2>requirements.txt</h2>
    <pre><code>pandas
numpy
matplotlib</code></pre>
  </div>

  <div class="section">
    <h2>部署与运行</h2>
    <p>
      克隆此仓库后，将以上两个文件放在同级目录，执行：
    </p>
    <pre><code>pip install -r requirements.txt
python report_generator.py</code></pre>
    <p>即可完成报告生成与雷达图展示。</p>
  </div>

</body>
</html>
