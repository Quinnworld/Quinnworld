const appearanceQuestions = [
  {
    id: "Q1",
    text: "你喜欢什么样的户外活动？",
    options: [
      { text: "海边日光浴", tags: { MC1R: 2, SLC24A5: 1, KITLG: 1 } },
      { text: "森林徒步", tags: { FGFR2: 1, TYR: 1, DCHS2: 1 } },
      { text: "极限运动", tags: { FTO: 1, MC4R: 1, RUNX2: 1 } },
      { text: "更喜欢室内", tags: { MC1R: -1, FTO: -1 } }
    ]
  },
  {
    id: "Q2",
    text: "你对高温/烈日的耐受度如何？",
    options: [
      { text: "很耐晒", tags: { MC1R: 2, SLC45A2: 2 } },
      { text: "一般", tags: { MC1R: 1, SLC45A2: 1 } },
      { text: "容易晒伤", tags: { MC1R: -2, KITLG: 1 } },
      { text: "基本不出门", tags: { FTO: -1 } }
    ]
  },
  {
    id: "Q3",
    text: "你对自己的头发最满意哪些方面？",
    options: [
      { text: "发质柔顺，易打理", tags: { EDAR: 2, FGFR2: 1 } },
      { text: "自然卷曲有型", tags: { EDAR: -2, FGFR3: 1, TBX15: 1 } },
      { text: "头发颜色独特", tags: { MC1R: 1, OCA2: 1, KITLG: 2 } },
      { text: "不太满意", tags: { EDAR: -1, MC1R: -1 } }
    ]
  },
  {
    id: "Q4",
    text: "你喜欢什么类型的饮食？",
    options: [
      { text: "高蛋白健身餐", tags: { FTO: -2, MC4R: -1 } },
      { text: "高碳水主食", tags: { FTO: 2, LEPR: 2 } },
      { text: "喜欢甜食", tags: { FTO: 2, MC4R: 1, TYR: 1 } },
      { text: "清淡素食", tags: { FTO: -1, SLC24A5: 1 } }
    ]
  },
  {
    id: "Q5",
    text: "你运动偏好？",
    options: [
      { text: "力量训练", tags: { RUNX2: 2, FGFR2: 1 } },
      { text: "有氧耐力", tags: { ADIPOQ: -1, FTO: -1 } },
      { text: "喜欢慢节奏", tags: { LEPR: 1 } },
      { text: "极少运动", tags: { FTO: 2 } }
    ]
  },
  {
    id: "Q6",
    text: "你最喜欢的季节？",
    options: [
      { text: "夏天", tags: { MC1R: 2, KITLG: 1 } },
      { text: "冬天", tags: { FTO: 1, MC4R: 1 } },
      { text: "春天", tags: { OCA2: 1, SLC24A5: 1 } },
      { text: "秋天", tags: { FGFR2: 1, TBX15: 1 } }
    ]
  },
  {
    id: "Q7",
    text: "你是否容易被蚊虫叮咬？",
    options: [
      { text: "非常容易", tags: { TYR: 2, KITLG: 2 } },
      { text: "偶尔", tags: { TYR: 1 } },
      { text: "很少", tags: { TYR: -1 } },
      { text: "几乎没有", tags: { TYR: -2 } }
    ]
  },
  {
    id: "Q8",
    text: "你喜欢什么风格的服饰？",
    options: [
      { text: "欧美时尚", tags: { MC1R: 1, OCA2: 1, FGFR2: 1 } },
      { text: "日韩清新", tags: { EDAR: 2, SLC24A5: 1 } },
      { text: "运动休闲", tags: { FTO: 1, MC4R: 1 } },
      { text: "民族风", tags: { KITLG: 1, PAX3: 1 } }
    ]
  },
  {
    id: "Q9",
    text: "你在社交中最常被夸奖什么？",
    options: [
      { text: "五官立体", tags: { FGFR2: 2, PAX3: 2, TBX15: 1 } },
      { text: "皮肤健康", tags: { MC1R: 2, SLC24A5: 1, OCA2: 1 } },
      { text: "气质出众", tags: { RUNX2: 1, FTO: 1 } },
      { text: "身材匀称", tags: { FTO: 2, MC4R: 2, LEPR: 1 } }
    ]
  },
  {
    id: "Q10",
    text: "你日常是否注重护肤？",
    options: [
      { text: "非常注重", tags: { MC1R: 2, SLC24A5: 1, TYR: 1 } },
      { text: "偶尔护肤", tags: { MC1R: 1, KITLG: 1 } },
      { text: "基本不护肤", tags: { MC1R: -1, TYR: -1 } },
      { text: "完全不在意", tags: { MC1R: -2, KITLG: -1 } }
    ]
  },
  {
    id: "Q11",
    text: "你笑起来时最突出的特点？",
    options: [
      { text: "酒窝明显", tags: { DCHS2: 2, RUNX2: 1 } },
      { text: "牙齿整齐", tags: { EDAR: 2, FGFR3: 1 } },
      { text: "颧骨分明", tags: { FGFR2: 1, TBX15: 2 } },
      { text: "脸型圆润", tags: { PAX3: 1, FTO: 1 } }
    ]
  },
  {
    id: "Q12",
    text: "你早上起床的状态？",
    options: [
      { text: "精力充沛", tags: { ADIPOQ: -1, FTO: -1 } },
      { text: "一般", tags: { ADIPOQ: 0 } },
      { text: "容易水肿", tags: { LEPR: 2, FTO: 2 } },
      { text: "很难起床", tags: { FTO: 2 } }
    ]
  },
  {
    id: "Q13",
    text: "你偏爱的发色或尝试过的发色？",
    options: [
      { text: "自然黑/棕", tags: { OCA2: 1, SLC45A2: 1 } },
      { text: "金色/浅色", tags: { MC1R: 2, SLC24A5: 2 } },
      { text: "红色/紫色", tags: { MC1R: 2, KITLG: 2 } },
      { text: "其他亮色", tags: { SLC24A5: 1, KITLG: 1 } }
    ]
  },
  {
    id: "Q14",
    text: "你平时喜欢的旅游体验？",
    options: [
      { text: "高原/山地", tags: { RUNX2: 2, FGFR2: 1 } },
      { text: "海岛/沙滩", tags: { MC1R: 2, OCA2: 1 } },
      { text: "都市文化", tags: { TBX15: 1, PAX3: 1 } },
      { text: "乡村田园", tags: { FGFR3: 1, ADIPOQ: 1 } }
    ]
  },
  {
    id: "Q15",
    text: "你在童年最显著的外貌变化？",
    options: [
      { text: "开始长雀斑", tags: { MC1R: 2, KITLG: 2 } },
      { text: "牙齿或颧骨变化", tags: { EDAR: 2, FGFR2: 1, TBX15: 1 } },
      { text: "体型明显改变", tags: { FTO: 2, LEPR: 1 } },
      { text: "基本没变化", tags: { RUNX2: -1, FTO: -1 } }
    ]
  }
];