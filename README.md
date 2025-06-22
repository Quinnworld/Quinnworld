<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>二次元世界 - 基因联动问卷系统</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(to right, #0f2027, #203a43, #2c5364);
      color: #fff;
      padding: 2em;
    }
    h1, h2 {
      text-align: center;
      margin-bottom: 1em;
    }
    .section {
      background-color: rgba(255, 255, 255, 0.1);
      padding: 1.5em;
      margin-bottom: 1.5em;
      border-radius: 10px;
    }
    label {
      display: block;
      margin-top: 1em;
    }
    input, select, textarea {
      width: 100%;
      padding: 0.5em;
      margin-top: 0.5em;
      border-radius: 5px;
      border: none;
    }
   : 0.5em;
    }
    button {
      margin-top: 1.5em;
      padding: 0.75em 2em;
      background-color: #3ac1c8;
      border: none;
      border-radius: 5px;
      font-size: 1em;
      color: #000;
      cursor: pointer;
    }
    #output {
      margin-top: 2em;
      background-color: rgba(255, 255, 255, 0.15);
      padding: 1em;
      border-radius: 10px;
    }
  </style>
</head>
<body>
  <h1>二次元世界</h1>
  <h2>基因联动问卷系统</h2>

  <div class="section" id="gene-questionnaire">
    <form id="geneForm">
      <!-- 问题 1: 基因样本选择 -->
      <label>请选择你的DNA基因样本类型：</label>
      <label><input type="radio" name="geneType" value="记录型" required> 记录型（传承过去的基因印记）</label>
      <label><input type="radio" name="geneType" value="感光型"> 感光型（对环境变化极为敏感）</label>
      <label><input type="radio" name="geneType" value="重组型"> 重组型（不断进化的混合体）</label>

      <!-- 问题 2: 细胞活跃指数 -->
      <label for="activityLevel">请选择你的细胞活跃指数：</label>
      <select name="activityLevel" id="activityLevel" required>
        <option value="">请选择</option>
        <option value="低活跃">低活跃（稳步运转）</option>
        <option value="中活跃">中活跃（适应进化）</option>
        <option value="高活跃">高活跃（极致爆发）</option>
      </select>

      <!-- 问题 3: 联动偏好 -->
      <label for="linkagePreference">请选择你的联动偏好（交叉效应）：</label>
      <select name="linkagePreference" id="linkagePreference" required>
        <option value="">请选择</option>
        <option value="情感共鸣">情感共鸣</option>
        <option value="科技融合">科技融合</option>
        <option value="神秘能量">神秘能量</option>
      </select>

      <!-- 问题 4: 基因描述 -->
      <label for="geneDescription">描述你内在的基因潜能与联动效应（可选）:</label>
      <textarea name="geneDescription" id="geneDescription" rows="4" placeholder="例如：深藏的记忆碎片碰撞出未来科技的火花..."></textarea>

      <button type="button" onclick="generateGeneEffect()">生成基因联动效应</button>
    </form>
  </div>

  <div id="output" class="section">
    <h3>生成结果：</h3>
    <div id="result">请填写问卷后点击按钮生成结果。</div>
  </div>

  <script>
    function generateGeneEffect() {
      const form = document.getElementById('geneForm');
      const formData = new FormData(form);
      const geneType = formData.get('geneType');
      const activityLevel = formData.get('activityLevel');
      const linkagePreference = formData.get('linkagePreference');
      const geneDescription = formData.get('geneDescription') || "无额外描述";

      // 模拟联动交叉效应：使用不同字段的首字母和随机因子拼接成一个联动基因代码
      const randomFactor = Math.floor(Math.random() * 1000);
      const geneCode = geneType.charAt(0) + activityLevel.charAt(0) + linkagePreference.charAt(0) + randomFactor;

      const resultHTML = `
        <p><strong>DNA基因样本类型：</strong> ${geneType}</p>
        <p><strong>细胞活跃指数：</strong> ${activityLevel}</p>
        <p><strong>联动偏好：</strong> ${linkagePreference}</p>
        <p><strong>个性基因描述：</strong> ${geneDescription}</p>
        <p><strong>生成的联动基因代码：</strong> ${geneCode}</p>
        <p><strong>联动效应解析：</strong> ${geneType}基因经过${activityLevel}状态激活，与${linkagePreference}偏好产生交叉联动，展现出独特的科学幻想特质。</p>
      `;

      document.getElementById('result').innerHTML = resultHTML;
    }
  </script>
</body>
</html>