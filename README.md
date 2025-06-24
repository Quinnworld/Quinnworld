<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>虚拟基因人格画像问卷（含付费解锁）</title>
  <style>
    html, body {
      height: 100%;
      margin: 0; padding: 0;
      background: #f5f7fa;
      font-family: "微软雅黑", sans-serif;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      padding: 30px 10px;
      box-sizing: border-box;
    }
    #mainbox {
      background: #fff;
      border-radius: 18px;
      box-shadow: 0 6px 24px #cdd2f1;
      width: 480px;
      max-width: 98vw;
      padding: 36px 24px 24px 24px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    h2, h3 {
      color: #2b2f4a;
      text-align: center;
      margin: 8px 0 24px;
      font-weight: 600;
      letter-spacing: 1.5px;
    }
    .question {
      width: 100%;
      margin-bottom: 18px;
    }
    .question-text {
      font-weight: 700;
      margin-bottom: 10px;
      font-size: 17px;
      color: #222;
    }
    .question-img {
      width: 100%;
      height: auto;
      border-radius: 8px;
      margin-bottom: 15px;
    }
    .option {
      background: #f3f3fa;
      border: 1.5px solid #c7d2fe;
      border-radius: 8px;
      padding: 13px 11px;
      margin: 8px 0;
      font-size: 16px;
      cursor: pointer;
      user-select: none;
      transition: all 0.15s ease;
      position: relative;
    }
    .option:hover {
      background: #d1d8fd;
    }
    .option.selected {
      background: linear-gradient(90deg,#818cf8 70%,#60a5fa 100%);
      color: white;
      border-color: #6366f1;
      font-weight: bold;
      box-shadow: 0 2px 8px #dbeafe;
    }
    #btnSubmit, #btnRestart {
      margin-top: 18px;
      padding: 14px;
      font-size: 18px;
      font-weight: 600;
      border-radius: 8px;
      border: none;
      background: linear-gradient(90deg,#6366f1,#60a5fa 70%);
      color: white;
      cursor: pointer;
      width: 100%;
      box-shadow: 0 2px 8px #dbeafe;
      transition: background 0.2s ease;
    }
    #btnSubmit:disabled {
      background: #dbeafe;
      cursor: not-allowed;
      color: #90a4c6;
    }
    #resultSection {
      display: none;
      width: 100%;
      margin-top: 24px;
    }
    #promptOutput {
      background: #f4f6fb;
      border-radius: 8px;
      padding: 16px;
      white-space: pre-wrap;
      font-size: 15.6px;
      line-height: 1.4;
      color: #333;
      max-height: 200px;
      overflow-y: auto;
