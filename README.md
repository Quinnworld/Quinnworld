const questionPool = [
  {
    id: "Q1", text: "你喜欢参加社交活动吗？",
    options: [
      { text: "非常不喜欢", tags: { extraversion: -2 } },
      { text: "不太喜欢", tags: { extraversion: -1 } },
      { text: "比较喜欢", tags: { extraversion: 1 } },
      { text: "非常喜欢", tags: { extraversion: 2 } }
    ]
  },
  {
    id: "Q2", text: "面对压力时你的情绪表现？",
    options: [
      { text: "非常紧张和焦虑", tags: { emotion_stability: -2, self_control: -1 } },
      { text: "有些不安", tags: { emotion_stability: -1, self_control: 0 } },
      { text: "情绪稳定", tags: { emotion_stability: 1, self_control: 1 } },
      { text: "非常冷静", tags: { emotion_stability: 2, self_control: 2 } }
    ]
  },
  {
    id: "Q3", text: "你喜欢尝试新鲜事物吗？",
    options: [
      { text: "完全不喜欢", tags: { novelty_seek: -2, openness: -1 } },
      { text: "不太喜欢", tags: { novelty_seek: -1, openness: 0 } },
      { text: "比较喜欢", tags: { novelty_seek: 1, openness: 1 } },
      { text: "非常喜欢", tags: { novelty_seek: 2, openness: 2 } }
    ]
  },
  {
    id: "Q4", text: "你通常是否按时完成任务？",
    options: [
      { text: "经常拖延", tags: { responsibility: -2 } },
      { text: "偶尔拖延", tags: { responsibility: -1 } },
      { text: "大部分时间按时", tags: { responsibility: 1 } },
      { text: "总是按时完成", tags: { responsibility: 2 } }
    ]
  },
  {
    id: "Q5", text: "你做决定时是否容易冲动？",
    options: [
      { text: "完全不会冲动", tags: { self_control: 2 } },
      { text: "不太冲动", tags: { self_control: 1 } },
      { text: "有时冲动", tags: { self_control: -1 } },
      { text: "非常冲动", tags: { self_control: -2 } }
    ]
  },
  {
    id: "Q6", text: "你是否喜欢探索未知？",
    options: [
      { text: "完全不喜欢", tags: { novelty_seek: -2 } },
      { text: "不太喜欢", tags: { novelty_seek: -1 } },
      { text: "比较喜欢", tags: { novelty_seek: 1 } },
      { text: "非常喜欢", tags: { novelty_seek: 2 } }
    ]
  },
  {
    id: "Q7", text: "你是否喜欢有条理地生活和工作？",
    options: [
      { text: "不喜欢", tags: { responsibility: -2, openness: -1 } },
      { text: "偶尔", tags: { responsibility: -1, openness: 0 } },
      { text: "经常", tags: { responsibility: 1, openness: 1 } },
      { text: "总是", tags: { responsibility: 2, openness: 2 } }
    ]
  },
  {
    id: "Q8", text: "你控制情绪的能力如何？",
    options: [
      { text: "很差", tags: { emotion_stability: -2, self_control: -2 } },
      { text: "一般", tags: { emotion_stability: -1, self_control: -1 } },
      { text: "较好", tags: { emotion_stability: 1, self_control: 1 } },
      { text: "非常好", tags: { emotion_stability: 2, self_control: 2 } }
    ]
  },
  {
    id: "Q9", text: "你是否愿意接受新观点和改变？",
    options: [
      { text: "完全不愿意", tags: { openness: -2 } },
      { text: "不太愿意", tags: { openness: -1 } },
      { text: "比较愿意", tags: { openness: 1 } },
      { text: "非常愿意", tags: { openness: 2 } }
    ]
  },
  {
    id: "Q10", text: "你是否喜欢团队合作？",
    options: [
      { text: "完全不喜欢", tags: { extraversion: -2, responsibility: -1 } },
      { text: "不太喜欢", tags: { extraversion: -1, responsibility: 0 } },
      { text: "比较喜欢", tags: { extraversion: 1, responsibility: 1 } },
      { text: "非常喜欢", tags: { extraversion: 2, responsibility: 2 } }
    ]
  },
  {
    id: "Q11", text: "你面对失败时的态度是？",
    options: [
      { text: "很沮丧，难以振作", tags: { emotion_stability: -2 } },
      { text: "有些消极", tags: { emotion_stability: -1 } },
      { text: "能较快调整", tags: { emotion_stability: 1 } },
      { text: "积极乐观", tags: { emotion_stability: 2 } }
    ]
  },
  {
    id: "Q12", text: "你是否喜欢规划未来？",
    options: [
      { text: "完全不喜欢", tags: { responsibility: -2, self_control: -1 } },
      { text: "不太喜欢", tags: { responsibility: -1, self_control: 0 } },
      { text: "比较喜欢", tags: { responsibility: 1, self_control: 1 } },
      { text: "非常喜欢", tags: { responsibility: 2, self_control: 2 } }
    ]
  }
];
