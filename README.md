<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>基因—个性报告自动化脚本</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    pre { background: #f5f5f5; padding: 10px; border-radius: 4px; overflow-x: auto; }
    code { font-family: Consolas, monospace; }
  </style>
</head>
<body>
  <h1>基因—个性报告自动化脚本（Python）</h1>
  <p>将下面代码保存为 <code>report_generator.py</code>，并确保已安装 <code>pandas</code>、<code>numpy</code>、<code>matplotlib</code> 等依赖。</p>
  <pre><code>
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import os

# ===== 1. 用户信息输入（示例：可替换为前端传参或文件读取） =====
def get_user_info():
    gender = input("请输入您的性别（male/female/other）：").strip().lower()
    if gender not in ['male','female','other']:
        gender = 'other'
    try:
        age = int(input("请输入您的年龄：").strip())
    except:
        age = 30
    return gender, age

gender, age = get_user_info()

# ===== 2. 初始化：六大维度名称及示例原始基因得分 =====
dimensions = ['Extraversion', 'Agreeableness', 'Conscientiousness',
              'Neuroticism', 'Openness', 'Stability']
initial_scores = np.array([75, 60, 82, 45, 90, 70])  # 示例，实际由PGS+G×G+概率推断

# ===== 3. 定义性别和年龄调整因子 =====
gender_factors = {
    'male':   [1.05, 0.95, 1.00, 1.10, 0.98, 1.00],
    'female': [0.95, 1.05, 1.02, 0.90, 1.02, 1.00]
}
def age_factor(age):
    # 将年龄映射为 [0.8,1.2] 区间
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
angles = np.linspace(0, 2*np.pi, len(dimensions), endpoint=False).tolist()
values = np.concatenate((adjusted_scores, [adjusted_scores[0]]))
angles += angles[:1]

fig, ax = plt.subplots(figsize=(6,6), subplot_kw=dict(polar=True))
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
    # TODO: 用实际内容替换占位符
    f.write(b"%PDF-1.4\n% 深度报告内容占位符\n")
print(f"\n深度报告已生成（付费下载）：{os.path.abspath(deep_report_path)}")
  </code></pre>

  <p>运行方式：</p>
  <ol>
    <li>在命令行执行：<code>python report_generator.py</code></li>
    <li>按提示输入性别和年龄。</li>
    <li>程序将弹出雷达图，并在终端输出初步报告；同时在脚本目录生成 <code>deep_report.pdf</code> 作为付费深度报告占位。</li>
  </ol>

  <p>您可根据需要：</p>
  <ul>
    <li>将原始 <code>initial_scores</code> 替换为真实模型输出。</li>
    <li>完善深度报告内容，如：基因-维度关联详情、后验概率可视化等。</li>
    <li>整合前端表单与支付接口，实现全流程自动化。</li>
  </ul>
</body>
</html>
