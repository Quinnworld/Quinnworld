// 显示结果区域
document.getElementById('radar-container').style.display = 'block';
document.getElementById('report').style.display = 'block';

// 更新雷达图
drawRadarChart(scores);

// 显示基础解析报告
const summary = generateSummary(scores);
document.getElementById('total-score').innerText = `综合总分：${summary.total} / 100`;
document.getElementById('summary-text').innerText = summary.desc;

// 显示深度解析报告
document.getElementById('detail-list').innerHTML = generateDetails(scores);

// 页面滚动到结果区域
document.getElementById('radar-container').scrollIntoView({ behavior: 'smooth' });
